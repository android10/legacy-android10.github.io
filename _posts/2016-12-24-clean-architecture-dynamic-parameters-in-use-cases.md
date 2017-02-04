---
id: 484
title: 'Clean Architecture&#8230;Dynamic Parameters in Use Cases'
date: 2016-12-24T22:05:11+00:00
author: fcejas
layout: post
guid: http://fernandocejas.com/?p=484
permalink: /2016/12/24/clean-architecture-dynamic-parameters-in-use-cases/
categories:
  - Android
  - Development
  - Java
  - Reactive Programming
  - Software Architecture
tags:
  - android
  - androiddev
  - architecture
  - clean architecture
  - dagger
  - dependency injection
  - development
  - java
  - mobile
  - mobile dev
  - programmer
  - programming
  - reactive
  - reactive programming
---
<span style="color: #333399;"><strong>Code is about evolution</strong></span>: A lot has been going on since my first approach of **<span style="color: #333399;">Clean Architecture on Android</span>** more than <span style="text-decoration: underline;">2 years ago</span>:

  * <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android&#8230; The clean way?.</a>
  * <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android&#8230; The evolution.</a>
  * <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">Tasting Dagger 2 on Android.</a>

<a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">The repo</a> has changed a lot, and people are still contributing and giving <a href="https://github.com/android10/Android-CleanArchitecture/issues" target="_blank">a lot of feedback</a>.
  
**<span style="color: #333399;">Kudos to the community!</span>**

## Why this article?

Everything started with <a href="https://github.com/android10/Android-CleanArchitecture/issues/32" target="_blank">a simple question on Github</a>:

<img class="aligncenter wp-image-496 size-large" src="http://fernandocejas.com/wp-content/uploads/2016/12/github_discussion-1024x912.png" alt="github_discussion" width="792" height="705" srcset="http://fernandocejas.com/wp-content/uploads/2016/12/github_discussion-1024x912.png 1024w, http://fernandocejas.com/wp-content/uploads/2016/12/github_discussion-300x267.png 300w, http://fernandocejas.com/wp-content/uploads/2016/12/github_discussion-768x684.png 768w, http://fernandocejas.com/wp-content/uploads/2016/12/github_discussion.png 1440w" sizes="(max-width: 792px) 100vw, 792px" />

I know that it was <span style="text-decoration: underline;">some time ago</span>, but I think it is worth to expose **<span style="color: #333399;">and share my idea with all of you here</span>**. As a plus, I really encourage you to look into all the feedback and discussions <a href="https://github.com/android10/Android-CleanArchitecture/issues" target="_blank">happening on github</a>, if you have not done it yet. <span style="color: #333399;"><strong>There is a lot of valuable information. </strong></span>Also, in order to better understand this writing, **<span style="color: #333399;">I recommend you to revisit those articles mentioned above.</span>**

## Use cases and dynamic parameters

In a nutshell and to put you in context, <span style="color: #333399;"><strong>the clean architecture example has 2 main use cases</strong></span>:

  * <a href="https://github.com/android10/Android-CleanArchitecture/blob/master/domain/src/main/java/com/fernandocejas/android10/sample/domain/interactor/GetUserList.java" target="_blank">GetUserList.java</a>
  * <a href="https://github.com/android10/Android-CleanArchitecture/blob/master/domain/src/main/java/com/fernandocejas/android10/sample/domain/interactor/GetUserDetails.java" target="_blank">GetUserDetails.java</a>

We are going to focus on **<span style="color: #333399;">&#8220;GetUserDetails&#8221;</span>** here. Why? Because this one <span style="text-decoration: underline;"><span style="color: #333399; text-decoration: underline;"><strong>needs a dynamic parameter</strong></span></span> **<span style="color: #333399;">(userId)</span>** to return the right data for an specific user and show it on the UI.

To facilitate understanding this in a easier way, <span style="text-decoration: underline;">look at the following picture as a reminder of the architecture</span>, where we have presenters which contain **<span style="color: #333399;">&#8220;UseCase&#8221;</span>** classes as collaborators:

<img class="aligncenter size-full wp-image-360" src="http://fernandocejas.com/wp-content/uploads/2015/07/clean_architecture_evolution.png" alt="clean_architecture_evolution" width="720" height="540" srcset="http://fernandocejas.com/wp-content/uploads/2015/07/clean_architecture_evolution.png 720w, http://fernandocejas.com/wp-content/uploads/2015/07/clean_architecture_evolution-300x225.png 300w" sizes="(max-width: 720px) 100vw, 720px" />

This is the original implementation of the already mentioned **<span style="color: #333399;">GetUserDetails.java</span>**:

<pre class="font-size:13 nums-toggle:false lang:java decode:true" title="GetUserDetails.java">/**
 * This class is an implementation of {@link UseCase} that represents a use case for
 * retrieving data related to an specific {@link User}.
 */
 public class GetUserDetails extends UseCase {
  
 	private final int userId;
    private final UserRepository userRepository;
  
    @Inject
 	public GetUserDetails(int userId, UserRepository userRepository,
 	    ThreadExecutor threadExecutor, PostExecutionThread postExecutionThread) {
      super(threadExecutor, postExecutionThread);
 	  this.userId = userId;
      this.userRepository = userRepository;
    }
  
 	@Override protected Observable buildUseCaseObservable() {
 	  return this.userRepository.user(this.userId);
    }
  }</pre>

**<span style="color: #333399;">As you can see, this &#8220;use case&#8221; inherits from a UseCase.java class</span>**, which by nature, makes sure that every single **<span style="color: #333399;">&#8220;UseCase&#8221; runs on a separate thread out of the Android main one</span>**, which is a good thing, <span style="text-decoration: underline;">remember?</span> We want to provide users with a smooth experience and thus, **<span style="color: #333399;">not overload the Android main UI thread.</span>**

There is a clear problem with this class design: <span style="text-decoration: underline;"><strong><span style="color: #333399; text-decoration: underline;">RIGIDNESS</span></strong></span>, which means that we are injecting the **<span style="color: #333399;">&#8220;userId&#8221;</span>** parameter in the constructor (<span style="text-decoration: underline;">at a Dagger module level</span>: <a href="https://github.com/android10/Android-CleanArchitecture/blob/master/presentation/src/main/java/com/fernandocejas/android10/sample/presentation/internal/di/modules/UserModule.java" target="_blank">UserModule.java</a>) which limits us in case we <span style="color: #333399;"><strong>do not know that variable value</strong></span> at the moment of our presenter instantiation.

In fact, this is a valid approach and works for this specific case but, what happens if we have a **<span style="color: #333399;">&#8220;LoginUserUseCase&#8221;</span>** for example? **<span style="color: #333399;">We suffer from the lack of that flexibility</span>** because we do not know upfront &#8220;username&#8221; and &#8220;password&#8221; when we instantiate our potential **<span style="color: #333399;">&#8220;LoginPresenter.java&#8221;</span>** class.

We need to f**<span style="color: #333399;">ind a simple way to pass in parameters when building the observable</span>** (through the abstract method **<span style="color: #333399;">buildUseCaseObservable()</span>**) which is responsible for emitting the items needed for our <span style="text-decoration: underline;">use case execution.</span>

This is the original method signature:

<pre class="font-size:13 nums-toggle:false lang:java decode:true" title="UseCase.java">/**
   * Builds an {@link Observable} which will be used when executing the current {@link UseCase}.
   */
  protected abstract Observable buildUseCaseObservable();</pre>

## Solution 1: the workaround

We can use <span style="color: #333399;"><strong>Optional<T></strong></span> to pass in parameters and change the signature of the method to look like as following:

<pre class="font-size:13 nums-toggle:false lang:java decode:true" title="UseCase.java">/**
   * Builds an {@link Observable} which will be used when executing the current {@link UseCase}.
   */
  protected abstract Observable buildUseCaseObservable(Optional&lt;Params&gt; params);</pre>

I consider this a VALID solution by using **<span style="color: #333399;">Optional<T></span>** for **<span style="color: #333399;">&#8220;Optional&#8221;</span>** parameters, that may be required or not for certain **<span style="color: #333399;">&#8220;UseCase&#8221;</span>** classes.
  
By the way, I&#8217;m not going to talk about <a href="http://fernandocejas.com/2016/02/20/how-to-use-optional-on-android-and-java/" target="_blank">the benefits of using Optional<T> on Android</a> since **<span style="color: #333399;">I have already done it in the past</span>**.

### GetUserDetails.java class implementation

This is now the final result of our modified **<span style="color: #333399;">&#8220;GetUserDetails&#8221;</span>** class:

<pre class="font-size:13 nums-toggle:false lang:java decode:true">/**
 * This class is an implementation of {@link UseCase} that represents a use case for
 * retrieving data related to an specific {@link User}.
 */
public class GetUserDetails extends UseCase {

  public static final String NAME = "userDetails";
  public static final String PARAM_USER_ID_KEY = "userId";

  @VisibleForTesting
  static final int PARAM_USER_ID_DEFAULT_VALUE = -1;

  private final UserRepository userRepository;

  @Inject
  public GetUserDetails(UserRepository userRepository, ThreadExecutor threadExecutor,
      PostExecutionThread postExecutionThread) {
    super(threadExecutor, postExecutionThread);
    this.userRepository = userRepository;
  }

  @Override protected Observable buildUseCaseObservable(Optional&lt;Params&gt; params) {
    if (params.isPresent()) {
      final int userId = params.get().getInt(PARAM_USER_ID_KEY, PARAM_USER_ID_DEFAULT_VALUE);
      return this.userRepository.user(userId);
    } else {
      return Observable.empty();
    }
  }
}</pre>

**<span style="color: #333399;">We got rid of our dynamic parameter in the constructor and we pass it in when we execute the &#8220;UseCase&#8221;.</span>** And by using **<span style="color: #333399;">Optional<Param></span>** we make sure that any client **<span style="color: #333399;">(UseCase)</span>** wanting to unwrap <span style="color: #333399;"><strong>&#8220;Params&#8221;</strong></span> will have to check their availability first. <a href="https://github.com/android10/Android-CleanArchitecture/commit/dfd61c29170b74298780ebb7cc991e0b2edbdfe6" target="_blank">Here is the commit</a> with all the modifications to the <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">repo</a> if you want to dive deeper.

### Params.java class implementation

**<span style="color: #333399;">This is no more than a class backed by a Map<T, K> which stores the parameters.</span>**
  
Basically I followed up the principles of the well known **<span style="color: #333399;">&#8220;Bundle.java&#8221;</span>** class on Android, but since I did not want to have any dependencies on the framework at a domain layer level, <span style="text-decoration: underline;">I opted for my own implementation.</span>

You may argue that this is **<span style="color: #333399;">reinventing the wheel</span>** but from my perspective, it is a very simple class (wrapper) with a clear purpose.

By the way, this is how it looks like:

<pre class="font-size:13 nums-toggle:false lang:java decode:true " title="Params.java">/**
 * Class backed by a Map, used to pass parameters to {@link UseCase} instances.
 */
public final class Params {
  public static final Params EMPTY = Params.create();

  private final Map&lt;String, Object&gt; parameters = new HashMap&lt;&gt;();

  private Params() {}

  public static Params create() {
    return new Params();
  }

  public void putInt(String key, int value) {
    parameters.put(key, value);
  }

  int getInt(String key, int defaultValue) {
    final Object object = parameters.get(key);
    if (object == null) {
      return defaultValue;
    }
    try {
      return (int) object;
    } catch (ClassCastException e) {
      return defaultValue;
    }
  }
}</pre>

You can grow this class or refactor it <span style="text-decoration: underline;">depending on your requirements/needs.</span>

## Solution 2: the power of refactoring

<a href="https://twitter.com/sockeqwe/status/812970762055352320?lang=en" target="_blank">After discussing with the community</a> I came up with a <span style="color: #000080;"><strong>more elegant solution</strong></span>.
  
It is worth mentioning that I wanted to keep the original one in this article to actually show **<span style="color: #000080;">how we can evolve our code by receiving constructive feedback and seeing how other professionals solve the same problems: The magic of refactoring.</span>**

This second approach consists of making our <a href="https://github.com/android10/Android-CleanArchitecture/blob/master/domain/src/main/java/com/fernandocejas/android10/sample/domain/interactor/UseCase.java" target="_blank">UseCase java class</a> generic, <span style="color: #000080;"><strong>which now requires 2 parameterized types:</strong></span>

  * **<span style="color: #000080;">T:</span>** The use case Observable<T> **<span style="color: #000080;">return type</span>**
  * <span style="color: #000080;"><strong>Param:</strong></span> A class type (inner in my implementation) which is specific to each use case, that will wrap <span style="color: #000080;"><strong>all the execution environment values</strong></span> required in order to execute our use case.

### UseCase.java class implementation

<pre class="font-size:13 nums-toggle:false lang:java decode:true" title="UseCase.java">public abstract class UseCase&lt;T, Params&gt; {
  ...

  /**
   * Builds an {@link Observable} which will be used when executing the current {@link UseCase}.
   */
  abstract Observable&lt;T&gt; buildUseCaseObservable(Params params);

  /**
   * Executes the current use case.
   *
   * @param observer {@link DisposableObserver} which will be listening to the observable build
   * by {@link #buildUseCaseObservable(Params)} ()} method.
   * @param params Parameters (Optional) used to build/execute this use case.
   */
  public void execute(DisposableObserver&lt;T&gt; observer, Params params) {
    Preconditions.checkNotNull(observer);
    final Observable&lt;T&gt; observable = this.buildUseCaseObservable(params)
        .subscribeOn(Schedulers.from(threadExecutor))
        .observeOn(postExecutionThread.getScheduler());
    addDisposable(observable.subscribeWith(observer));
  }

  ...
}</pre>

**<span style="color: #000080;">The main benefit of this is that you get better semantics because our inner &#8220;param&#8221; class can contain its properly named fields, getters and setters. </span>**

### GetUserDetails.java class implementation

<pre class="font-size:13 nums-toggle:false lang:java decode:true" title="GetUserDetails.java">/**
 * This class is an implementation of {@link UseCase} that represents a use case for
 * retrieving data related to an specific {@link User}.
 */
public class GetUserDetails extends UseCase&lt;User, GetUserDetails.Params&gt; {

  private final UserRepository userRepository;

  @Inject
  GetUserDetails(UserRepository userRepository, ThreadExecutor threadExecutor,
      PostExecutionThread postExecutionThread) {
    super(threadExecutor, postExecutionThread);
    this.userRepository = userRepository;
  }

  @Override Observable&lt;User&gt; buildUseCaseObservable(Params params) {
    Preconditions.checkNotNull(params);
    return this.userRepository.user(params.userId);
  }

  public static final class Params {

    private final int userId;

    private Params(int userId) {
      this.userId = userId;
    }

    public static Params forUser(int userId) {
      return new Params(userId);
    }
  }
}</pre>

### GetUserList.java class implementation

**<span style="color: #000080;">There is another question that comes to my mind: What happens with UseCase with empty parameters? </span>**We can just use <span style="color: #000080;"><strong>&#8220;Void&#8221;</strong></span> as **<span style="color: #000080;">&#8220;Param&#8221;</span>** type. For example, our <span style="color: #000080;"><strong>&#8220;GetUserList&#8221;</strong></span> use case which does not require any extra data to be executed:

<pre class="font-size:13 nums-toggle:false lang:java decode:true" title="GetUserList.java">/**
 * This class is an implementation of {@link UseCase} that represents a use case for
 * retrieving a collection of all {@link User}.
 */
public class GetUserList extends UseCase&lt;List&lt;User&gt;, Void&gt; {

  private final UserRepository userRepository;

  @Inject
  GetUserList(UserRepository userRepository, ThreadExecutor threadExecutor,
      PostExecutionThread postExecutionThread) {
    super(threadExecutor, postExecutionThread);
    this.userRepository = userRepository;
  }

  @Override Observable&lt;List&lt;User&gt;&gt; buildUseCaseObservable(Void unused) {
    return this.userRepository.users();
  }
}</pre>

Again, <a href="https://github.com/android10/Android-CleanArchitecture/commit/aaa21c1b7cb4dc7b80a4da7f0552036bcb7ff40e" target="_blank">here is the commit</a> if you want to see all the changes involved.

## Conclusion

**<span style="color: #000080;">Here you have 2 simple approaches to use dynamic parameters in Android clean architecture use cases.</span>**
  
Remember that there are <span style="color: #000080;"><strong>NO SILVER BULLETS</strong></span> and you can agree or not with it, but the most important thing is to use <span style="color: #000080;"><strong>WHAT WORKS FOR YOU BASED ON YOUR REQUIREMENTS</strong></span> (Respect software engineering good practices and principles).

Also some advice:

  * <span style="color: #000080;"><strong>You establish the RULES for your framework.</strong></span>
  * <span style="color: #000080;"><strong>Do not overthink too much.</strong></span>
  * <span style="color: #000080;"><strong>Do not put a lot of overhead, start simple and move towards complexity.</strong></span>
  * <span style="color: #000080;"><strong>Be open to get constructive feedback.</strong></span>

<span style="color: #000080;"><strong>My original implementation worked</strong></span> but it turned out to not be as **FLEXIBLE** as I wanted to: the more requirements we have the more we need to evolve our code base. Writing this article was a pending debt to me, but as I love to say: <span style="color: #000080;"><strong>&#8220;better late than never&#8221;</strong></span>.
  
<a href="https://github.com/android10/Android-CleanArchitecture/issues/32" target="_blank">Please leave any feedback in the discussions section on Github</a>.

## Links

  * <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Clean Architecture on Android Repo.</a>
  * <a href="https://github.com/android10/Android-CleanArchitecture/issues" target="_blank">Clean Architecture discussions.</a>
  * <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android&#8230; The clean way?.</a>
  * <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android&#8230; The evolution.</a>
  * <a href="http://fernandocejas.com/2016/02/20/how-to-use-optional-on-android-and-java/" target="_blank">How to use Optional<T> on Android and Java.</a>
  * <a href="https://github.com/android10/Android-CleanArchitecture/issues" target="_blank">Clean Architecture Discussions.</a>