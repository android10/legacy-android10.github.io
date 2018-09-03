---
id: 2
title: Unit testing asynchronous methods with Mockito
date: 2014-04-08T00:32:44+00:00
author: fernando
description: Unit testing asynchronous methods with Mockito
layout: post
permalink: /2014/04/08/unit-testing-asynchronous-methods-with-mockito/
image: assets/images/asynchronous_testing_mockito.jpeg
comments: false
featured: false
hidden: false
categories: [ java, threading, programming, testing ]
tags:
  - acceptance
  - fake object
  - integration
  - junit
  - mock
  - mockito
  - powermock
  - stub
  - test
  - test double
  - testing
---
After promising that I would be writing and maintaining my blog, here I go again (3289423987 attempt). So in this occasion I want to write about <a href="http://static.javadoc.io/org.mockito/mockito-core/2.21.0/org/mockito/Mockito.html" target="_blank">Mockito</a>... yes, this mocking framework that is a "must have" when writing unit tests ;).


## Introduction

This is article assumes that you know what a <a href="http://en.wikipedia.org/wiki/Unit_testing" target="_blank">unit test</a> is and why you should write tests in general. 

Also I strongly recommend <a href="http://martinfowler.com/articles/mocksArentStubs.html" target="_blank">this famous article from Martin Fowler</a> that talks about test doubles, **it is a must read to understand about test doubles.**


## Common case scenario

**Sometimes we have to test methods that use callbacks, meaning that they are asynchronous by definition.** 

These methods are not easy to test and using Thread.sleep(milliseconds) method to wait for the response is not a good practice and can convert your tests in <a href="http://martinfowler.com/articles/nonDeterminism.html" target="_blank">non-deterministic</a> ones (I have seen this many times to be honest).
  
So how do we do this? <a href="https://site.mockito.org/" target="_blank">Mockito</a> to the rescue!


## Let's see an example

Suppose we have a class called **DummyCaller** that implements a **DummyCallback** and has a method ```doSomethingAsynchronously()``` that delegates its functionality to a collaborator of the the class called **DummyCollaborator** that has a ```doSomethingAsynchronously(DummyCallback callback)``` as well, but receives a callback as a parameter (in this case our **DummyCallback**), so this methods creates a new thread to run his job and then gives us a result when is done.
  
Here is the code to understand this scenario in a better way:

```java
public interface DummyCallback {
  public void onSuccess(List<String> result);
  public void onFail(int code);
}
```

```java
public class DummyCaller implements DummyCallback {
  private final DummyCollaborator dummyCollaborator;

  private List<String> result = new ArrayList<String>();

  public DummyCaller(DummyCollaborator dummyCollaborator) {
    this.dummyCollaborator = dummyCollaborator;
  }

  public void doSomethingAsynchronously() {
    dummyCollaborator.doSomethingAsynchronously(this);
  }

  public List<String> getResult() {
    return this.result;
  }

  @Override
  public void onSuccess(List<String> result) {
    this.result = result;
    System.out.println("On success");
  }

  @Override
  public void onFail(int code) {
    System.out.println("On Fail");
  }
}
```

```java
public class DummyCollaborator {
  public static int ERROR_CODE = 1;

  public DummyCollaborator() {
    // empty
  }

  public void doSomethingAsynchronously (final DummyCallback callback) {
    new Thread(new Runnable() {
      @Override
      public void run() {
        try {
          Thread.sleep(5000);
          callback.onSuccess(Collections.EMPTY_LIST);
        } catch (InterruptedException e) {
          callback.onFail(ERROR_CODE);
          e.printStackTrace();
        }
      }
    }).start();
  }
}
```


## Creating our test class

**We have 2 options to test our asynchronous method** but first we will create our test class **DummyCollaboratorCallerTest** (for convention we just add Test at the end of the class so this becomes part of its name).

```java
public class DummyCollaboratorCallerTest {
  // Class under test
  private DummyCaller dummyCaller;

  @Mock
  private DummyCollaborator mockDummyCollaborator;

  @Captor
  private ArgumentCaptor<DummyCallback> dummyCallbackArgumentCaptor;

  @Before
  public void setUp() {
    MockitoAnnotations.initMocks(this);
    dummyCaller = new DummyCaller(mockDummyCollaborator);
  }
}
```

So here we are using **MockitoAnotations** to initialize both <a href="https://static.javadoc.io/org.mockito/mockito-core/2.2.28/org/mockito/MockitoAnnotations.html" target="_blank">Mock</a> and <a href="https://static.javadoc.io/org.mockito/mockito-core/2.6.9/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a>, but do not worry about them yet, cause this is what we will be seeing next.
  
The only thing to take into account here is that both mock and class under test are being initialized before each test is executed in the ```setUp()``` method (using the ```@Before``` annotation).
  
Remember that for unit testing all the collaborators for a **CUT (class under test)** must be test doubles. 

Let's take a look at our 2 test solutions going forward.


## Setting up an answer for our callback

This is our test case using a <a href="https://static.javadoc.io/org.mockito/mockito-core/2.21.0/org/mockito/Mockito.html#do_family_methods_stubs" target="_blank">doAnswer()</a> for stubbing a method with a generic **Answer**. 

This means that since we need a callback to return immediately (synchronously), we generate an answer so when the method under test is called, the callback will be executed right away with the data we tell it to return.
  
**Finally we call our real method and verify state and interaction.**

```java
@Test
  public void testDoSomethingAsynchronouslyUsingDoAnswer() {
    final List<String> results = Arrays.asList("One", "Two", "Three");
    // Let's do a synchronous answer for the callback
    doAnswer(new Answer() {
      @Override
      public Object answer(InvocationOnMock invocation) throws Throwable {
        ((DummyCallback)invocation.getArguments()[0]).onSuccess(results);
        return null;
      }
    }).when(mockDummyCollaborator).doSomethingAsynchronously(
        any(DummyCallback.class));

    // Let's call the method under test
    dummyCaller.doSomethingAsynchronously();

    // Verify state and interaction
    verify(mockDummyCollaborator, times(1)).doSomethingAsynchronously(
        any(DummyCallback.class));
    assertThat(dummyCaller.getResult(), is(equalTo(results)));
  }
```


## Using an ArgumentCaptor

Our second option is to use an <a href="https://static.javadoc.io/org.mockito/mockito-core/2.6.9/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a>.
 
Here we treat our callback asynchronously: we capture the **DummyCallback** object passed to our **DummyCollaborator** using an <a href="https://static.javadoc.io/org.mockito/mockito-core/2.6.9/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a>.
  
Finally we can make all our assertions at the test method level and call ```onSuccess()``` when we want to verify state and interaction.

```java
@Test
  public void testDoSomethingAsynchronouslyUsingArgumentCaptor() {
    // Let's call the method under test
    dummyCaller.doSomethingAsynchronously();

    final List<String> results = Arrays.asList("One", "Two", "Three");

    // Let's call the callback. ArgumentCaptor.capture() works like a matcher.
    verify(mockDummyCollaborator, times(1)).doSomethingAsynchronously(
        dummyCallbackArgumentCaptor.capture());

    // Some assertion about the state before the callback is called
    assertThat(dummyCaller.getResult().isEmpty(), is(true));

    // Once you're satisfied, trigger the reply on callbackCaptor.getValue().
    dummyCallbackArgumentCaptor.getValue().onSuccess(results);

    // Some assertion about the state after the callback is called
    assertThat(dummyCaller.getResult(), is(equalTo(results)));
  }
```


## Conclusion

The main difference between both solutions is that when using <a href="https://static.javadoc.io/org.mockito/mockito-core/2.21.0/org/mockito/Mockito.html#do_family_methods_stubs" target="_blank">DoAnswer()</a> **we are creating an anonymous inner class, and casting (in an unsafe way)** the elements from ```invocation.getArguments()[n]``` to the data type we want, but in case we modify our parameters the test will "fail fast" letting know that something has happened. 

On the other side, when using <a href="https://static.javadoc.io/org.mockito/mockito-core/2.6.9/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a> we probably **have more control** cause we can call the callbacks in the order we want in case we need it.
  
As interest in unit testing, this is a common case that sometimes we do not know how deal with, so in my experience using both solutions has helped me to have a robust approach when having to test asynchronous methods.
  
I hope you find this article useful, and as always, remember that any feedback is very welcome. Of course if you have any doubt do not hesitate to contact me.


## Code Sample

<a href="https://github.com/android10/Inside_Android_Testing" target="_blank">Here is the link where you can find this example and others.</a> 

Most of them are related to Java and Android because this comes from a talk I gave a couple of months ago (the slides are in english but the presentation in spanish):

<p><center><script async class="speakerdeck-embed" data-id="196436502c600131664d26c7baad6ac3" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script></center>

<p><center><iframe width="640" height="480" src="//www.youtube.com/embed/I0qDmbwGz3o" frameborder="0" allowfullscreen=""></iframe></center>


## Further Reading

I highly recommend to take a look at <a href="https://site.mockito.org/" target="_blank">Mockito documentation</a> to have a better understanding of the framework. The documentation is very clear and has great examples.