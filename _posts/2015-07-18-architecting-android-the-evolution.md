---
id: 7
title: 'Architecting Android...The evolution'
date: 2015-07-18T16:18:56+00:00
author: Fernando Cejas
layout: post
permalink: /2015/07/18/architecting-android-the-evolution/
categories:
  - Android
  - Development
  - Java
  - Reactive Programming
  - Software Architecture
  - Testing
tags:
  - android
  - androiddev
  - architecture
  - clean
  - clean architecture
  - code injection
  - developer
  - developers
  - development
  - junit
  - mock
  - mockito
  - model view presenter
  - mvp
  - observable
  - patterns
  - programming
  - repository
  - repository pattern
  - strategy pattern
  - test
  - testing
---
<p class="justify"><span class="boldtext">Hey there!</span> After a while (and a lot of feedback received) I decided it was a good time to get back to this topic and <span class="boldtext">give you another taste of what I consider a good approach when it comes to architecting modern mobile applications (android in this case).</span></p>

<p class="justify">Before getting started, I assume that <span class="boldtext"><a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">you already read my previous post about Architecting Android…The clean way?</a></span> If not, this is a good opportunity to get in touch with it in order to have a better understanding of the story I’m going to tell you right here:</p>

<img class="aligncenter wp-image-208 size-full" src="/assets/migrated/clean_architecture1.png" alt="clean_architecture" width="647" height="440" srcset="/assets/migrated/clean_architecture1.png 647w, /assets/migrated/clean_architecture1-300x204.png 300w" sizes="(max-width: 647px) 100vw, 647px" />

## Architecture evolution

<p class="justify"><span class="boldtext">Evolution stands for a gradual process in which something changes into a different and usually more complex or better form.</span></p>

<p class="justify">Said that, software evolves and changes over the time and indeed an architecture. <span class="boldtext">Actually a good software design must help us grow and extend our solution by keeping it healthy without having to rewrite everything</span> (although there are cases where this approach is better, but that is a topic for another article, so let’s focus in what I pointed out earlier, trust me).</p>

<p class="justify"><span class="boldtext">In this article,</span> I am going to walk you through key points I consider necessary and important, <span class="boldtext">to keep the sanity of our android codebase.</span> Keep in mind this picture and let’s get started.</p>

<img class="aligncenter size-full wp-image-206" src="/assets/migrated/clean_architecture_android.png" alt="clean_architecture_android" width="673" height="292" srcset="/assets/migrated/clean_architecture_android.png 673w, /assets/migrated/clean_architecture_android-300x130.png 300w" sizes="(max-width: 673px) 100vw, 673px" />

## Reactive approach: RxJava

<p class="justify">I’m not going to talk about the benefits of <span class="boldtext">RxJava</span> here (<a href="https://github.com/ReactiveX/RxJava/wiki/" target="_blank">I assume you already had a taste of it</a>), since <a href="http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/" target="_blank">there are a lot articles</a> and <a href="https://speakerdeck.com/benjchristensen" target="_blank">badasses</a> of this technology that are doing an excellent job out there. However, <span class="boldtext">I will point out what makes it interesting in regards of android applications development, and how it has helped me evolve my first approach of clean architecture.</span></p>

<p class="justify">First, I opted for a reactive pattern by converting use cases (called <span class="boldtext">interactors</span> in the clean architecture naming convention) <span class="boldtext">to return Observables&lt;T&gt; which means all the lower layers will follow the chain and return Observables&lt;T&gt; too.</span></p>

```java
public abstract class UseCase {

  private final ThreadExecutor threadExecutor;
  private final PostExecutionThread postExecutionThread;

  private Subscription subscription = Subscriptions.empty();

  protected UseCase(ThreadExecutor threadExecutor,
      PostExecutionThread postExecutionThread) {
    this.threadExecutor = threadExecutor;
    this.postExecutionThread = postExecutionThread;
  }

  protected abstract Observable buildUseCaseObservable();

  public void execute(Subscriber UseCaseSubscriber) {
    this.subscription = this.buildUseCaseObservable()
        .subscribeOn(Schedulers.from(threadExecutor))
        .observeOn(postExecutionThread.getScheduler())
        .subscribe(UseCaseSubscriber);
  }

  public void unsubscribe() {
    if (!subscription.isUnsubscribed()) {
      subscription.unsubscribe();
    }
  }
}
```

<p class="justify">As you can see here, all use cases inherit from this abstract class and implement the abstract method <span class="boldtext">buildUseCaseObservable()</span> which will setup an <span class="boldtext">Observable&lt;T&gt;</span> that is going to do the hard job and return the needed data.</p>

<p class="justify"><span class="boldtext">Something to highlight is the fact that on execute() method, we make sure our Observable&lt;T&gt; executes itself in a separate thread</span>, thus, minimizing how much we block the android main thread. The result is push back on the Android main thread through the android main thread scheduler.</p>

<p class="justify"><span class="boldtext">So far, we have our Observable&lt;T&gt; up and running, but, as you know, someone has to observe the data sequence emitted by it.</span> To achieve this, I evolved presenters (part of <span class="boldtext">MVP</span> in the presentation layer) into <span class="boldtext">Subscribers</span> which would <span class="boldtext">“react”</span> to these emitted items by use cases, in order to update the user interface.</p>

<p class="justify">Here is how the subscriber looks like:</p>

```java
private final class UserListSubscriber extends DefaultSubscriber<List<User>> {

  @Override public void onCompleted() {
    UserListPresenter.this.hideViewLoading();
  }

  @Override public void onError(Throwable e) {
    UserListPresenter.this.hideViewLoading();
    UserListPresenter.this.showErrorMessage(new DefaultErrorBundle((Exception) e));
    UserListPresenter.this.showViewRetry();
  }

  @Override public void onNext(List<User> users) {
    UserListPresenter.this.showUsersCollectionInView(users);
  }
}
```

<p class="justify">Every subscriber is an inner class inside each presenter and implements a <span class="boldtext">DefaultSubscriber&lt;T&gt;</span> created basically for default error handling.</p>

<p class="justify"><span class="boldtext">After putting all pieces in place,</span> you can get the whole idea by having a look at the following picture:</p>

<img class="aligncenter size-full wp-image-360" src="/assets/migrated/clean_architecture_evolution.png" alt="clean_architecture_evolution" width="720" height="540" srcset="/assets/migrated/clean_architecture_evolution.png 720w, /assets/migrated/clean_architecture_evolution-300x225.png 300w" sizes="(max-width: 720px) 100vw, 720px" />

<p class="justify">Let’s enumerate a bunch of benefits we get out of <span class="boldtext">this RxJava based approach:</span></p>

* <p class="justify"><span class="boldtext">Decoupling between Observables and Subscribers:</span> makes maintainability and testing easier.</p>
* <p class="justify"><span class="boldtext">Simplified asynchronous tasks:</span> java threads and futures are complex to manipulate and synchronize if more than one single level of asynchronous execution is required, so by using schedulers we can jump between background and main thread in an easy way (with no extra effort), especially when we need to update the UI. We also avoid what we call a “callback hell”, which makes our code unreadable and hard to follow up.</p>
* <p class="justify"><span class="boldtext">Data transformation/composition:</span> we can combine multiple Observables<T>  without affecting the client, which makes our solution more scalable.</p>
* <p class="justify"><span class="boldtext">Error handling:</span> a signal is emitted to the consumer when an error has occurred within any Observable&lt;T&gt;.</p>

<p class="justify">From my point of view there is one drawback, and indeed a price to pay, which has to do with <span class="boldtext">the learning curve for developers who are not familiar with the concept.</span> However, you get very valuable stuff out of it. <span class="boldtext">Reactive for the win!</span></p>

## Dependency Injection: Dagger 2

<p class="justify">I’m not going to talk much of <span class="boldtext">dependency injection cause <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">I have already written a whole article</a>, which I strongly recommend you to read,</span> so we can stay on the same page here.</p>

<p class="justify">With that being said, it is worth mentioning, that by implementing a dependency injection framework like <span class="boldtext">Dagger 2 </span>we gain:</p>

* <p class="justify"><span class="boldtext">Components reuse</span>, since dependencies can be injected and configured externally.</p>
* <p class="justify">When injecting abstractions as collaborators, we can just change the implementation of any object without having to make a lot of changes in our codebase, since that <span class="boldtext">object instantiation resides in one place isolated and decoupled</span>.</p>
* <p class="justify">Dependencies can be injected into a component: <span class="boldtext">it is possible to inject mock implementations of these dependencies which makes testing easier.</span></p>

## Lambda expressions: Retrolambda

<p class="justify"><span class="boldtext">No one will complain about making use of Java 8 lambdas in our code</span>,  and even more when they simplify it and get rid of a lot of boilerplate, as you can see in this piece of code:</p>

```java
private final Action1<UserEntity> saveToCacheAction =
    userEntity -> {
      if (userEntity != null) {
        CloudUserDataStore.this.userCache.put(userEntity);
      }
    };
```

<p class="justify"><span class="boldtext">However, I have mixed feelings here and will explain why</span>. It turns out that at <a href="https://developers.soundcloud.com/blog/" target="_blank">@SoundCloud</a> we had a discussion around <a href="https://github.com/orfjackal/retrolambda" target="_blank">Retrolambda</a>, <span class="boldtext">mainly whether or not to use it,</span> and the outcome was:</p>

* <span class="boldtext">Pros:</span>
  * Lambdas and method references.
  * Try with resources.
  * Dev karma.

* <span class="boldtext">Cons:</span>   
  * Accidental use of Java 8 APIs.
  * 3rd part lib, quite intrusive.
  * 3rd part gradle plugin to make it work with Android.

<p class="justify">Finally we decided it was not something that would solve any problems for us: <span class="boldtext">your code looks better and more readable but it was something we could live without, since nowadays all the most powerful IDEs contain code folding options which cover this need, at least in an acceptable manner.</span></p>

<p class="justify">Honestly, the main reason why I used it here, was more to play around it and have a taste of lambdas on Android, although <span class="boldtext">I would probably use it again for a spare time project.</span> I will leave the decision up to you. I am just exposing my field of vision here. <span class="boldtext">Of course the <a href="https://github.com/orfjackal" target="_blank">author</a> of this library deserves my kudos for such an amazing job.</span></p>

## Testing approach

<p class="justify">In terms of testing, <span class="boldtext">not big changes</span> in relation with the first version of the example:</p>

* <span class="boldtext">Presentation layer:</span> UI tests with Espresso 2 and Android Instrumentation.
* <span class="boldtext">Domain layer:</span> JUnit + Mockito since it is a regular Java module.
* <span class="boldtext">Data layer:</span> Migrated test battery to use Robolectric 3 + JUnit + Mockito. Tests for this layer used to live in a separate Android Module, since back then (at the moment of the first version of the example), <span class="boldtext">there was no built-in unit test support and setting up a framework like robolectric was complicated and required a serie of hacks to make it work properly.</span>

<p class="justify">
  Fortunately that <span class="boldtext">is part of the past </span>and now everything works out of the box so I could relocated them inside the data module, specifically into its default test location: <span class="boldtext">src/test/java</span> folder.
</p>

## Package organization

<p class="justify">I consider code/package organization one of the key factors of a good architecture: <span class="boldtext">package structure is the very first thing encountered by a programmer when browsing source code. Everything flows from it. Everything depends on it.</span></p>

<p class="justify">We can distinguish between <span class="boldtext">2 paths</span> you can take to divide up your application into packages:</p>

* <p class="justify"><span class="boldtext">Package by layer:</span> Each package contains items that usually are not closely related to each other. This results in packages with low cohesion and low modularity, with high coupling between packages. As a result, editing a feature involves editing files across different packages. In addition, deleting a feature can almost never be performed in a single operation.</p>
* <p class="justify"><span class="boldtext">Package by feature:</span> It uses packages to reflect the feature set. It tries to place all items related to a single feature (and only that feature) into a single package. This results in packages with high cohesion and high modularity, and with minimal coupling between packages. Items that work closely together are placed next to each other. They are not spread out all over the application.</p>

<p class="justify"><span class="boldtext">My recommendation is to go with packages by features</span>, which bring these main benefits:</p>

* <span class="boldtext">Higher Modularity</span>
* <span class="boldtext">Easier Code Navigation</span>
* <span class="boldtext">Minimizes Scope</span>

<p class="justify">It is also interesting to add that if you are working with <span class="boldtext">feature teams</span> (as we do at <a href="https://twitter.com/soundcloud" target="_blank">@SoundCloud</a>), <span class="boldtext">code ownership will be easier to organize and more modularized</span>, which is a win in a growing organization where many developers work on the same codebase.</p>

<img class="aligncenter wp-image-368" src="/assets/migrated/package_organization-795x1024.png" alt="package_organization" width="450" height="580" srcset="/assets/migrated/package_organization-795x1024.png 795w, /assets/migrated/package_organization-233x300.png 233w, /assets/migrated/package_organization.png 916w" sizes="(max-width: 450px) 100vw, 450px" />

<p class="justify">As you can see, my approach looks like packages organized by layer: <span class="boldtext">I might have gotten wrong here (and group everything under &#8216;users&#8217; for example)</span> but I will <span class="boldtext">forgive myself in this case,</span> because this sample is for learning purpose and what I wanted to expose, were the main concepts of the clean architecture approach. <span class="boldtext">DO AS I SAY, NOT AS I DO :).</span></p>

## Extra ball: organizing your build logic

<p class="justify"><span class="boldtext">We all know that you build a house from the foundations up.</span> The same happens with software development, and here I want to remark that, from my perspective, <span class="boldtext">the build system (and its organization) is an important piece of a software architecture.</span></p>

<p class="justify">On Android, we use gradle, which is a platform agnostic build system and indeed, very powerful. The idea here is to go through a <span class="boldtext">bunch of tips and tricks</span> that can simplify your life when it comes to how organize the way you build your application:</p>

* <span class="boldtext">Group stuff by functionality in separate gradle build files.</span>

<img class="aligncenter wp-image-372" src="/assets/migrated/gradle_organization-283x300.png" alt="gradle_organization" width="210" height="223" srcset="/assets/migrated/gradle_organization-283x300.png 283w, /assets/migrated/gradle_organization.png 428w" sizes="(max-width: 210px) 100vw, 210px" />

```groovy
def ciServer = 'TRAVIS'
def executingOnCI = "true".equals(System.getenv(ciServer))

// Since for CI we always do full clean builds, we don't want to pre-dex
// See http://tools.android.com/tech-docs/new-build-system/tips
subprojects {
  project.plugins.whenPluginAdded { plugin ->
    if ('com.android.build.gradle.AppPlugin'.equals(plugin.class.name) ||
        'com.android.build.gradle.LibraryPlugin'.equals(plugin.class.name)) {
      project.android.dexOptions.preDexLibraries = !executingOnCI
    }
  }
}
```

```groovy
apply from: 'buildsystem/ci.gradle'
apply from: 'buildsystem/dependencies.gradle'

buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:1.2.3'
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
  }
}

allprojects {
  ext {
	...
  }
}
...
```

<p class="justify">Thus, you can use <span class="boldtext">"apply from: 'buildsystem/ci.gradle'"</span> to plug that configuration to any gradle build file. <span class="boldtext">Do not put everything on only one build.gradle file otherwise you will start creating a monster. Lesson learned.</span></p>

* <span class="boldtext">Create maps of dependencies</span>

```groovy
...

ext {
  //Libraries
  daggerVersion = '2.0'
  butterKnifeVersion = '7.0.1'
  recyclerViewVersion = '21.0.3'
  rxJavaVersion = '1.0.12'

  //Testing
  robolectricVersion = '3.0'
  jUnitVersion = '4.12'
  assertJVersion = '1.7.1'
  mockitoVersion = '1.9.5'
  dexmakerVersion = '1.0'
  espressoVersion = '2.0'
  testingSupportLibVersion = '0.1'
  
  ...
  
  domainDependencies = [
      daggerCompiler:     "com.google.dagger:dagger-compiler:${daggerVersion}",
      dagger:             "com.google.dagger:dagger:${daggerVersion}",
      javaxAnnotation:    "org.glassfish:javax.annotation:${javaxAnnotationVersion}",
      rxJava:             "io.reactivex:rxjava:${rxJavaVersion}",
  ]

  domainTestDependencies = [
      junit:              "junit:junit:${jUnitVersion}",
      mockito:            "org.mockito:mockito-core:${mockitoVersion}",
  ]

  ...

  dataTestDependencies = [
      junit:              "junit:junit:${jUnitVersion}",
      assertj:            "org.assertj:assertj-core:${assertJVersion}",
      mockito:            "org.mockito:mockito-core:${mockitoVersion}",
      robolectric:        "org.robolectric:robolectric:${robolectricVersion}",
  ]
}
```

```groovy
apply plugin: 'java'

sourceCompatibility = 1.7
targetCompatibility = 1.7

...

dependencies {
  def domainDependencies = rootProject.ext.domainDependencies
  def domainTestDependencies = rootProject.ext.domainTestDependencies

  provided domainDependencies.daggerCompiler
  provided domainDependencies.javaxAnnotation

  compile domainDependencies.dagger
  compile domainDependencies.rxJava

  testCompile domainTestDependencies.junit
  testCompile domainTestDependencies.mockito
}
```

<p class="justify"><span class="boldtext">This is very useful if you wanna reuse the same artifact version across different modules in your project</span>, or maybe the other way around, where you have to apply different dependency versions to different modules. Another plus one, is that <span class="boldtext">you also control the dependencies in one place</span> and, for instance, bumping an artifact version is pretty straightforward.</p>

## Wrapping up

<p class="justify">That is pretty much I have for now, and as a conclusion, keep in mind <span class="boldtext">there are no silver bullets</span>. <span class="boldtext">However, a good software architecture will help us keep our code clean and healthy, as well as scalable and easy to maintain.</span></p>

<p class="justify">There is a few more things I would like to point out and they have to do with attitudes you should take when facing a software problem:</p>

* <span class="boldtext">Respect SOLID principles.</span>
* <span class="boldtext">Do not over think (do not do over engineering).</span>
* <span class="boldtext">Be pragmatic.</span>
* <span class="boldtext">Minimize framework (android) dependencies in your project as much as you can.</span>

## Source code

  * <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Clean architecture github repository - master branch</a>
  * <a href="https://github.com/android10/Android-CleanArchitecture/releases" target="_blank">Clean architecture github repository - releases</a>

## Further reading:

  * <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android..the clean way</a>
  * <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">Tasting Dagger 2 on Android</a>
  * <a href="https://speakerdeck.com/android10/the-mayans-lost-guide-to-rxjava-on-android" target="_blank">The Mayans Lost Guide to RxJava on Android</a>
  * <a href="https://speakerdeck.com/android10/it-is-about-philosophy-culture-of-a-good-programmer" target="_blank">It is about philosophy: Culture of a good programmer</a>

<center><script class="speakerdeck-embed" src="//speakerdeck.com/assets/embed.js" async="" data-id="f706afb4728549f187682c855ab53163" data-ratio="1.33333333333333"></script></center>

<center><script class="speakerdeck-embed" src="//speakerdeck.com/assets/embed.js" async="" data-id="75eeca307fa10132c8d45e43b27e95d0" data-ratio="1.33333333333333"></script></center>

## References

  * <a href="https://github.com/ReactiveX/RxJava/wiki" target="_blank">RxJava wiki by Netflix</a>
  * <a href="https://blog.8thlight.com/uncle-bob/2014/05/11/FrameworkBound.html" target="_blank">Framework bound by Uncle Bob</a>
  * <a href="https://docs.gradle.org/current/userguide/userguide.html" target="_blank">Gradle user guide</a>
  * <a href="http://www.javapractices.com/topic/TopicAction.do?Id=205" target="_blank">Package by feature, not layer</a>