---
id: 2
title: Unit testing asynchronous methods with Mockito
date: 2014-04-08T00:32:44+00:00
author: Fernando Cejas
layout: post
permalink: /2014/04/08/unit-testing-asynchronous-methods-with-mockito/
categories:
  - Development
  - Testing
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
<p class="justify">After promising that I would be writing and maintaining my blog, here I go again (3289423987 attempt). So in this occasion I want to write about <a href="https://code.google.com/p/mockito/" target="_blank">Mockito</a>... yes, this mocking framework that is a <span class="boldtext">"must have"</span> when writing unit tests ;).</p>

## Introduction

<p class="justify">This is article assumes that you know what a <a href="http://en.wikipedia.org/wiki/Unit_testing" target="_blank">unit test</a> is and why you <span class="boldtext">should write tests.</span> Also I strongly recommend <a href="http://martinfowler.com/articles/mocksArentStubs.html" target="_blank">this famous article from Martin Fowler</a> that talks about test doubles, it is a must read to understand about test doubles.</p>

## Common case scenario

<p class="justify">Sometimes we have to test methods that use callbacks, meaning that they are asynchronous by definition. These methods are not easy to test and using <span class="boldtext">Thread.sleep(milliseconds)</span> method to wait for the response is not a good practice and can convert your tests in <a href="http://martinfowler.com/articles/nonDeterminism.html" target="_blank">non-deterministic</a> ones (I have seen this many times to be honest).</p>
  
<p class="justify">So how do we do this? <a href="https://code.google.com/p/mockito/" target="_blank">Mockito</a> to the rescue!</p>

## Let's see an example

<p class="justify">Suppose we have a class called <span class="boldtext">DummyCaller</span> that implements a <span class="boldtext">DummyCallback</span> and has a method <span class="boldtext">doSomethingAsynchronously()</span> that delegates its functionality to a collaborator of the the class called <span class="boldtext">DummyCollaborator</span> that has a <span class="boldtext">doSomethingAsynchronously(DummyCallback callback)</span> as well, but receives a callback as a parameter (in this case our <span class="boldtext">DummyCallback</span>), so this methods creates a new thread to run his job and then gives us a result when is done.</p>
  
<p class="justify">Here is the code to understand this scenario in a better way:</p>

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

<p class="justify">We have 2 options to test our asynchronous method but first we will create our test class <span class="boldtext">DummyCollaboratorCallerTest</span> (for convention we just add Test at the end of the class so this becomes part of its name).</p>

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

<p class="justify">So here we are using <span class="boldtext">MockitoAnotations</span> to initialize both <a href="http://docs.mockito.googlecode.com/hg/org/mockito/MockitoAnnotations.html" target="_blank">Mock</a> and <a href="http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a>, but do not worry about them yet, cause this is what we will be seeing next.</p>
  
<p class="justify">The only thing to take into account here is that both mock and class under test are being initialized before each test is executed in the <span class="boldtext">setUp()</span> method (using the <span class="boldtext">@Before</span> annotation).</p>
  
<p class="justify">Remember that for unit testing all the collaborators for a CUT (class under test) must be test doubles. Let's take a look at our 2 test solutions going forward.</p>

## Setting up an answer for our callback

<p class="justify">This is our test case using a <a href="http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html" target="_blank">doAnswer()</a> for stubbing a method with a generic Answer. This means that since we need a callback to return immediately (synchronously), we generate an answer so when the method under test is called, the callback will be executed right away with the data we tell it to return.</p>
  
<p class="justify">Finally we call our real method and verify state and interaction.</p>

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

<p class="justify">Our second option is to use an <a href="http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a>. Here we treat our callback asynchronously: we capture the <span class="boldtext">DummyCallback</span> object passed to our <span class="boldtext">DummyCollaborator</span> using an <a href="http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a>.</p>
  
<p class="justify">Finally we can make all our assertions at the test method level and call <span class="boldtext">onSuccess()</span> when we want to verify state and interaction.</p>

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

<p class="justify">The main difference between both solutions is that when using <a href="http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html" target="_blank">DoAnswer()</a> we are creating an anonymous inner class, and casting (in an unsafe way) the elements from <span class="boldtext">invocation.getArguments()[n]</span> to the data type we want, but in case we modify our parameters the test will "fail fast" letting know that something has happened. On the other side, when using <a href="http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a> we probably have more control cause we can call the callbacks in the order we want in case we need it.</p>
  
<p class="justify">As interest in unit testing, this is a common case that sometimes we do not know how deal with, so in my experience using both solutions has helped me to have a robust approach when having to test asynchronous methods.</p>
  
<p class="justify">I hope you find this article useful, and as always, remember that any feedback is very welcome. Of course if you have any doubt do not hesitate to contact me.</p>

## Code Sample

<p class="justify"><a href="https://github.com/android10/Inside_Android_Testing" target="_blank">Here is the link where you can find this example and others.</a> Most of them are related to Java and Android because this comes from a talk I gave a couple of months ago (the slides are in english but the presentation in spanish):</p>

<p><center><script async class="speakerdeck-embed" data-id="196436502c600131664d26c7baad6ac3" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script></center></p>

<p><center><iframe width="640" height="480" src="//www.youtube.com/embed/I0qDmbwGz3o" frameborder="0" allowfullscreen=""></iframe></center></p>

## Further Reading

<p class="justify">I highly recommend to take a look at <a href="http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html" target="_blank">Mockito documentation</a> to have a better understanding of the framework. The documentation is very clear and has great examples. See you!</p>