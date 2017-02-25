---
id: 11
title: 'Clean Architecture: Dynamic Parameters in Use Cases'
date: 2016-12-24T22:05:11+00:00
author: Fernando Cejas
description: Using dynamic parameters in a clean architecture approach for android
layout: post
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
<p class="justify"><span class="boldtext">Code is about evolution</span>: A lot has been going on since my first approach of <span class="boldtext">Clean Architecture on Android</span> more than <span class="underlinetext">2 years ago:</span></p>

  * <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android...The clean way?.</a>
  * <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android...The evolution.</a>
  * <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">Tasting Dagger 2 on Android.</a>

<p class="justify"><a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">The repo</a> has changed a lot, and people are still contributing and giving <a href="https://github.com/android10/Android-CleanArchitecture/issues" target="_blank">a lot of feedback</a>.</p>
  
<p class="justify"><span class="boldtext">Kudos to the community!</span></p>

## Why this article?

<p class="justify">Everything started with <a href="https://github.com/android10/Android-CleanArchitecture/issues/32" target="_blank">a simple question on Github</a>:</p>

<!-- Legacy image from migration -->
<img class="aligncenter wp-image-496 size-large" src="/assets/migrated/github_discussion-1024x912.png" alt="github_discussion" width="792" height="705" srcset="/assets/migrated/github_discussion-1024x912.png 1024w, /assets/migrated/github_discussion-300x267.png 300w, /assets/migrated/github_discussion-768x684.png 768w, /assets/migrated/github_discussion.png 1440w" sizes="(max-width: 792px) 100vw, 792px" />

<p class="justify">I know that it was <span class="underlinetext">some time ago</span>, but I think it is worth to expose <span class="boldtext">and share my idea with all of you here</span>. As a plus, I really encourage you to look into all feedback and discussions <a href="https://github.com/android10/Android-CleanArchitecture/issues" target="_blank">happening on github,</a> if you have not done it yet. <span class="boldtext">There is a lot of valuable information. </span>Also, in order to better understand this writing, <span class="boldtext">I recommend you to revisit those articles mentioned above.</span></p>

## Use cases and dynamic parameters

<p class="justify">In a nutshell and to put you in context, <span class="boldtext">the clean architecture example has 2 main use cases</span>:</p>

  * <a href="https://github.com/android10/Android-CleanArchitecture/blob/master/domain/src/main/java/com/fernandocejas/android10/sample/domain/interactor/GetUserList.java" target="_blank">GetUserList.java</a>
  * <a href="https://github.com/android10/Android-CleanArchitecture/blob/master/domain/src/main/java/com/fernandocejas/android10/sample/domain/interactor/GetUserDetails.java" target="_blank">GetUserDetails.java</a>

<p class="justify">We are going to focus on <span class="boldtext">"GetUserDetails"</span> here. Why? Because this one <span class="underlinetext"><span class="boldtext">needs a dynamic parameter</span></span> <span class="boldtext">(userId)</span> to return the right data for an specific user and show it on the UI.</p>

<p class="justify">To facilitate understanding this in a easier way, <span class="underlinetext">look at the following picture as a reminder of the architecture</span>, where we have presenters which contain <span class="boldtext">"UseCase"</span> classes as collaborators:</p>

<!-- Legacy image from migration -->
<img class="aligncenter size-full wp-image-360" src="/assets/migrated/clean_architecture_evolution.png" alt="clean_architecture_evolution" width="720" height="540" srcset="/assets/migrated/clean_architecture_evolution.png 720w, /assets/migrated/clean_architecture_evolution-300x225.png 300w" sizes="(max-width: 720px) 100vw, 720px" />

<p class="justify">This is the original implementation of the already mentioned <span class="boldtext">GetUserDetails.java</span>:</p>

```java
/**
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
}
```

<p class="justify"><span class="boldtext">As you can see, this "use case" inherits from a UseCase.java class</span>, which by nature, makes sure that every single <span class="boldtext">"UseCase" runs on a separate thread out of the Android main one</span>, which is a good thing, <span class="underlinetext">remember?</span> We want to provide users with a smooth experience and thus, <span class="boldtext">not overload the Android main UI thread.</span></p>

<p class="justify">There is a clear problem with this class design: <span class="boldtext">RIGIDNESS</span>, which means that we are injecting the <span class="boldtext">"userId"</span> parameter in the constructor (<span class="underlinetext">at a Dagger module level</span>: <a href="https://github.com/android10/Android-CleanArchitecture/blob/master/presentation/src/main/java/com/fernandocejas/android10/sample/presentation/internal/di/modules/UserModule.java" target="_blank">UserModule.java</a>) which limits us in case we <span class="boldtext">do not know that variable value</span> at the moment of our presenter instantiation.</p>

<p class="justify">In fact, this is a valid approach and works for this specific case but, what happens if we have a <span class="boldtext">"LoginUserUseCase"</span> for example? <span class="boldtext">We suffer from the lack of that flexibility</span> because we do not know upfront "username" and "password" when we instantiate our potential <span class="boldtext">"LoginPresenter.java"</span> class.</p>

<p class="justify">We need to <span class="boldtext">find a simple way to pass in parameters when building the observable</span> (through the abstract method <span class="boldtext">buildUseCaseObservable()</span>) which is responsible for emitting the items needed for our <span class="underlinetext">use case execution.</span></p>

<p class="justify">This is the original method signature:</p>

```java
 /**
  * Builds an {@link Observable} which will be used when executing the current {@link UseCase}.
  */
  protected abstract Observable buildUseCaseObservable();
```

## Solution 1: the workaround

<p class="justify">We can use <span class="boldtext">"Optional&lt;T&gt;"</span> to pass in parameters and change the signature of the method to look like as following:</p>

```java
  /**
   * Builds an {@link Observable} which will be used when executing the current {@link UseCase}.
   */
  protected abstract Observable buildUseCaseObservable(Optional<Params> params);
```

<p class="justify">I consider this a VALID solution by using <span class="boldtext">Optional&lt;T&gt;</span> for <span class="boldtext">"Optional"</span> parameters, that may be required or not for certain <span class="boldtext">"UseCase"</span> classes.</p>
  
<p class="justify">By the way, I'm not going to talk about <a href="http://fernandocejas.com/2016/02/20/how-to-use-optional-on-android-and-java/" target="_blank">the benefits of using Optional&lt;T&gt; on Android</a> since <span class="boldtext">I have already done it in the past</span>.</p>

### GetUserDetails.java class implementation

<p class="justify">This is now the final result of our modified <span class="boldtext">"GetUserDetails"</span> class:</p>

```java
/**
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
}
```

<p class="justify"><span class="boldtext">We got rid of our dynamic parameter in the constructor and we pass it in when we execute the "UseCase".</span> And by using <span class="boldtext">Optional&lt;Param&gt;</span> we make sure that any client <span class="boldtext">(UseCase)</span> wanting to unwrap <span class="boldtext">"Params"</span> will have to check their availability first. <a href="https://github.com/android10/Android-CleanArchitecture/commit/dfd61c29170b74298780ebb7cc991e0b2edbdfe6" target="_blank">Here is the commit</a> with all the modifications to the <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">repo</a> if you want to dive deeper.</p>

### Params.java class implementation

<p class="justify"><span class="boldtext">This is no more than a class backed by a Map<T, K> which stores the parameters.</span></p>
  
<p class="justify">Basically I followed up the principles of the well known <span class="boldtext">"Bundle.java"</span> class on Android, but since I did not want to have any dependencies on the framework at a domain layer level, <span class="underlinetext">I opted for my own implementation.</span></p>

<p class="justify">You may argue that this is <span class="boldtext">reinventing the wheel</span> but from my perspective, it is a very simple class (wrapper) with a clear purpose.</p>

<p class="justify">By the way, this is how it looks like:</p>

```java
/**
 * Class backed by a Map, used to pass parameters to {@link UseCase} instances.
 */
public final class Params {
  public static final Params EMPTY = Params.create();

  private final Map<String, Object> parameters = new HashMap<>();

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
}
```

<p class="justify">You can grow this class or refactor it <span class="underlinetext">depending on your requirements/needs.</span></p>

## Solution 2: the power of refactoring

<p class="justify"><a href="https://twitter.com/sockeqwe/status/812970762055352320?lang=en" target="_blank">After discussing with the community</a> I came up with a <span class="boldtext">more elegant solution</span>.</p>
  
<p class="justify">It is worth mentioning that I wanted to keep the original one in this article to actually show <span class="boldtext">how we can evolve our code by receiving constructive feedback and seeing how other professionals solve the same problems: The magic of refactoring.</span></p>

<p class="justify">This second approach consists of making our <a href="https://github.com/android10/Android-CleanArchitecture/blob/master/domain/src/main/java/com/fernandocejas/android10/sample/domain/interactor/UseCase.java" target="_blank">UseCase java class</a> generic, <span class="boldtext">which now requires 2 parameterized types:</span></p>

  * <span class="boldtext">T:</span> The use case Observable<T> <span class="boldtext">return type</span>
  * <span class="boldtext">Param:</span> A class type (inner in my implementation) which is specific to each use case, that will wrap <span class="boldtext">all the execution environment values</span> required in order to execute our use case.

### UseCase.java class implementation

```java
public abstract class UseCase<T, Params> {
  ...

  /**
   * Builds an {@link Observable} which will be used when executing the current {@link UseCase}.
   */
  abstract Observable<T> buildUseCaseObservable(Params params);

  /**
   * Executes the current use case.
   *
   * @param observer {@link DisposableObserver} which will be listening to the 
   * observable build by {@link #buildUseCaseObservable(Params)} ()} method.
   * @param params Parameters (Optional) used to build/execute this use case.
   */
  public void execute(DisposableObserver<T> observer, Params params) {
    Preconditions.checkNotNull(observer);
    final Observable<T> observable = this.buildUseCaseObservable(params)
        .subscribeOn(Schedulers.from(threadExecutor))
        .observeOn(postExecutionThread.getScheduler());
    addDisposable(observable.subscribeWith(observer));
  }

  ...
}
```

<p class="justify"><span class="boldtext">The main benefit of this is that you get better semantics because our inner "param" class can contain its properly named fields, getters and setters. </span></p>

### GetUserDetails.java class implementation

```java
/**
 * This class is an implementation of {@link UseCase} that represents a use case for
 * retrieving data related to an specific {@link User}.
 */
public class GetUserDetails extends UseCase<User, GetUserDetails.Params> {

  private final UserRepository userRepository;

  @Inject
  GetUserDetails(UserRepository userRepository, ThreadExecutor threadExecutor,
      PostExecutionThread postExecutionThread) {
    super(threadExecutor, postExecutionThread);
    this.userRepository = userRepository;
  }

  @Override Observable<User> buildUseCaseObservable(Params params) {
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
}
```

### GetUserList.java class implementation

<p class="justify"><span class="boldtext">There is another question that comes to my mind: What happens with UseCase with empty parameters? </span>We can just use <span class="boldtext">"Void"</span> as <span class="boldtext">"Param"</span> type. For example, our <span class="boldtext">"GetUserList"</span> use case which does not require any extra data to be executed:</p>

```java
/**
 * This class is an implementation of {@link UseCase} that represents a use case for
 * retrieving a collection of all {@link User}.
 */
public class GetUserList extends UseCase<List<User>, Void>; {

  private final UserRepository userRepository;

  @Inject
  GetUserList(UserRepository userRepository, ThreadExecutor threadExecutor,
      PostExecutionThread postExecutionThread) {
    super(threadExecutor, postExecutionThread);
    this.userRepository = userRepository;
  }

  @Override Observable<List<User>> buildUseCaseObservable(Void unused) {
    return this.userRepository.users();
  }
}
```

<p class="justify">Again, <a href="https://github.com/android10/Android-CleanArchitecture/commit/aaa21c1b7cb4dc7b80a4da7f0552036bcb7ff40e" target="_blank">here is the commit</a> if you want to see all the changes involved.</p>

## Conclusion

<p class="justify"><span class="boldtext">Here you have 2 simple approaches to use dynamic parameters in Android clean architecture use cases.</span></p>
  
<p class="justify">Remember that there are <span class="boldtext">NO SILVER BULLETS</span> and you can agree or not with it, but the most important thing is to use <span class="boldtext">WHAT WORKS FOR YOU BASED ON YOUR REQUIREMENTS</span> (Respect software engineering good practices and principles).</p>

Also some advice:

  * <span class="boldtext">You establish the RULES for your framework.</span>
  * <span class="boldtext">Do not overthink too much.</span>
  * <span class="boldtext">Do not put a lot of overhead, start simple and move towards complexity.</span>
  * <span class="boldtext">Be open to get constructive feedback.</span>

<p class="justify"><span class="boldtext">My original implementation worked</span> but it turned out to not be as FLEXIBLE as I wanted to: the more requirements we have the more we need to evolve our code base. Writing this article was a pending debt to me, but as I love to say: <span class="boldtext">"better late than never"</span>.</p>
  
<p class="justify"><a href="https://github.com/android10/Android-CleanArchitecture/issues/32" target="_blank">Please leave any feedback in the discussions section on Github</a>.</p>

## Links

  * <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Clean Architecture on Android Repo.</a>
  * <a href="https://github.com/android10/Android-CleanArchitecture/issues" target="_blank">Clean Architecture discussions.</a>
  * <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android&#8230; The clean way?.</a>
  * <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android&#8230; The evolution.</a>
  * <a href="http://fernandocejas.com/2016/02/20/how-to-use-optional-on-android-and-java/" target="_blank">How to use Optional<T> on Android and Java.</a>
  * <a href="https://github.com/android10/Android-CleanArchitecture/issues" target="_blank">Clean Architecture Discussions.</a>