---
id: 91
title: Unit testing asynchronous methods with Mockito
date: 2014-04-08T00:32:44+00:00
author: fcejas
layout: post
guid: http://fernandocejas.com/?p=91
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
After promising (<del>and not keeping my promise</del>) that I would be writing and maintaining my blog, here I go again (**3289423987 attempt**). But lest&#8217;s forget about that&#8230;
  
So in this occasion I wanted to write about <a href="https://code.google.com/p/mockito/" target="_blank">Mockito</a>&#8230;yes, this mocking framework that is a **<span style="color: #009933;">&#8216;must have&#8217;</span>** when writing your unit tests ;).

### **Introduction**

This is article assumes that you know what a <a href="http://en.wikipedia.org/wiki/Unit_testing" target="_blank">unit test</a> is and why you **<span style="color: #009933;">should write tests.</span>**
  
Also I strongly recommend <a href="http://martinfowler.com/articles/mocksArentStubs.html" target="_blank">this famous article from Martin Fowler</a> that talks about test doubles, it is a must read to understand about test doubles.

### **Common case scenario**

Sometimes we have to test methods that use callbacks, meaning that they are asynchronous by definition. These methods are not easy to test and using **<span style="color: #009933;">Thread.sleep(milliseconds)</span>** method to wait for the response is not a good practice and can convert your tests in <a href="http://martinfowler.com/articles/nonDeterminism.html" target="_blank">non-deterministic</a> ones (I have seen this many times to be honest).
  
So how do we do this? <a href="https://code.google.com/p/mockito/" target="_blank">Mockito</a> to the rescue!

### **Let&#8217;s put an example**

Suppose we have a class called **<span style="color: #009933;">DummyCaller</span>** that implements a **<span style="color: #009933;">DummyCallback</span>** and has a method **<span style="color: #009933;">doSomethingAsynchronously()</span>** that delegates its functionality to a collaborator of the the class called **<span style="color: #009933;">DummyCollaborator</span>** that has a **<span style="color: #009933;">doSomethingAsynchronously(DummyCallback callback)</span>** as well, but receives a callback as a parameter (in this case our **<span style="color: #009933;">DummyCallback</span>**), so this methods creates a new thread to run his job and then gives us a result when is done.
  
Here is the code to understand this scenario in a better way:

<pre class="theme:github font-size:14 lang:java decode:true">public interface DummyCallback {
	public void onSuccess(List&lt;String&gt; result);
	public void onFail(int code);
}</pre>

<pre class="theme:github font-size:14 lang:java decode:true">public class DummyCaller implements DummyCallback {

	private final DummyCollaborator dummyCollaborator;

	private List&lt;String&gt; result = new ArrayList&lt;String&gt;();

	public DummyCaller(DummyCollaborator dummyCollaborator) {
		this.dummyCollaborator = dummyCollaborator;
	}

	public void doSomethingAsynchronously() {
		dummyCollaborator.doSomethingAsynchronously(this);
	}

	public List&lt;String&gt; getResult() {
		return this.result;
	}

	@Override
	public void onSuccess(List&lt;String&gt; result) {
		this.result = result;
		System.out.println("On success");
	}

	@Override
	public void onFail(int code) {
		System.out.println("On Fail");
	}
}</pre>

<pre class="theme:github font-size:14 lang:java decode:true">public class DummyCollaborator {

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
}</pre>

### **Creating our test class**

We have 2 options to test our asynchronous method but first we will create our test class **<span style="color: #009933;">DummyCollaboratorCallerTest</span>** (for convention we just add Test at the end of the class so this becomes part of its name).

<pre class="theme:github font-size:14 lang:java decode:true">public class DummyCollaboratorCallerTest {

	// Class under test
	private DummyCaller dummyCaller;

	@Mock
	private DummyCollaborator mockDummyCollaborator;

	@Captor
	private ArgumentCaptor&lt;DummyCallback&gt; dummyCallbackArgumentCaptor;

	@Before
	public void setUp() {
		MockitoAnnotations.initMocks(this);
		dummyCaller = new DummyCaller(mockDummyCollaborator);
	}
}</pre>

So here we are using **<span style="color: #009933;">MockitoAnotations</span>** to initialize both <a href="http://docs.mockito.googlecode.com/hg/org/mockito/MockitoAnnotations.html" target="_blank">Mock</a> and <a href="http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a>, but don&#8217;t worry about them yet, cause this is what we will be seeing next.
  
The only thing to take into account here is that both mock and class under test are being initialized before each test is executed in the **<span style="color: #009933;">setUp()</span>** method (using the **<span style="color: #009933;">@Before</span>** annotation).
  
Remember that for unit testing all the collaborators for a CUT (class under test) must be test doubles.
  
Let&#8217;s take a look at our 2 test solutions.

### **Setting up an answer for our callback**

This is our test case using a <a href="http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html" target="_blank">doAnswer()</a> for stubbing a method with a generic Answer. This means that since we need a callback to return immediately (synchronously), we generate an answer so when the method under test is called, the callback will be executed right away with the data we tell it to return.
  
Finally we call our real method and verify state and interaction.

<pre class="theme:github font-size:14 lang:java decode:true">@Test
	public void testDoSomethingAsynchronouslyUsingDoAnswer() {
		final List&lt;String&gt; results = Arrays.asList("One", "Two", "Three");
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
	}</pre>

### **Using an ArgumentCaptor**

Our second option is to use an <a href="http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a>. Here we treat our callback asynchronously: we capture the **<span style="color: #009933;">DummyCallback</span>** object passed to our **<span style="color: #009933;">DummyCollaborator</span>** using an <a href="http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a>.
  
Finally we can make all our assertions at the test method level and call **<span style="color: #009933;">onSuccess()</span>** when we want to verify state and interaction.

<pre class="theme:github font-size:14 lang:java decode:true">@Test
	public void testDoSomethingAsynchronouslyUsingArgumentCaptor() {
		// Let's call the method under test
		dummyCaller.doSomethingAsynchronously();

		final List&lt;String&gt; results = Arrays.asList("One", "Two", "Three");

		// Let's call the callback. ArgumentCaptor.capture() works like a matcher.
		verify(mockDummyCollaborator, times(1)).doSomethingAsynchronously(
				dummyCallbackArgumentCaptor.capture());

		// Some assertion about the state before the callback is called
		assertThat(dummyCaller.getResult().isEmpty(), is(true));

		// Once you're satisfied, trigger the reply on callbackCaptor.getValue().
		dummyCallbackArgumentCaptor.getValue().onSuccess(results);

		// Some assertion about the state after the callback is called
		assertThat(dummyCaller.getResult(), is(equalTo(results)));
	}</pre>

### **Conclusion**

The main difference between both solutions is that when using <a href="http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html" target="_blank">DoAnswer()</a> we are creating an anonymous inner class, and casting (in an unsafe way) the elements from **<span style="color: #009933;">invocation.getArguments()[n]</span>** to the data type we want, but in case we modify our parameters the test will &#8216;fail fast&#8217; letting know that something has happened. On the other side, when using <a href="http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html" target="_blank">ArgumentCaptor</a> we probably have more control cause we can call the callbacks in the order we want in case we need it.
  
As interest in unit testing, this is a common case that sometimes we do not know how deal with, so in my experience using both solutions has helped me to have a robust approach when having to test asynchronous methods.
  
I hope you find this article useful, and as always, remember that any feedback is very welcome, as well as other ways of doing this. Of course if you have any doubt do not hesitate to contact me.

### **Code Sample**

[Here is the link where you can find this example and others](https://github.com/android10/Inside_Android_Testing). Most of them are related with Java and Android because this comes from a talk I gave a couple of months ago.
  
The presentation is in english but the video is in spanish (sorry for those who do not understand my argentinian accent&#8230;haha&#8230;BTW I will try to upload an english version as soon as possible&#8230;)



<center>
</center>

### **Further Reading**

I highly recommend to take a look at <a href="http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html" target="_blank">Mockito documentation</a> to have a better understanding of the framework. The documentation is very clear and has great examples. See you!