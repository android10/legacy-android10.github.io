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
<p style="text-align: justify;">
  Hey there! After a while (and a lot of feedback received) I decided it was a good time to get back to this topic and <strong><span style="color: #333399;">give you another taste of what I consider a good approach when it comes to architecting modern mobile applications (android in this case).</span></strong>
</p>

<p style="text-align: justify;">
  Before getting started, I assume that <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">you already read my previous post about Architecting Android…The clean way?</a> If not, this is a good opportunity to get in touch with it in order to have a better understanding of the story I’m going to tell you right here:
</p>

[<img class="aligncenter wp-image-208 size-full" src="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture1.png" alt="clean_architecture" width="647" height="440" srcset="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture1.png 647w, http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture1-300x204.png 300w" sizes="(max-width: 647px) 100vw, 647px" />](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture1.png)

### Architecture evolution

<p style="text-align: justify;">
  <strong><span style="color: #333399;">Evolution stands for a gradual process in which something changes into a different and usually more complex or better form.</span></strong>
</p>

<p style="text-align: justify;">
  Said that, software evolves and changes over the time and indeed an architecture. <strong><span style="color: #333399;">Actually a good software design must help us grow and extend our solution by keeping it healthy</span></strong> without having to rewrite everything (although there are cases where this approach is better, but that is a topic for another article, so let’s focus in what I pointed out earlier, trust me).
</p>

<p style="text-align: justify;">
  In this article, I am going to walk you through key points I consider necessary and important, <span style="color: #333399;"><strong>to keep the sanity of our android codebase.</strong></span> Keep in mind this picture and let’s get started.
</p>

[<img class="aligncenter size-full wp-image-206" src="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_android.png" alt="clean_architecture_android" width="673" height="292" srcset="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_android.png 673w, http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_android-300x130.png 300w" sizes="(max-width: 673px) 100vw, 673px" />](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_android.png)

### Reactive approach: RxJava

<p style="text-align: justify;">
  I’m not going to talk about the benefits of RxJava here (<a href="https://github.com/ReactiveX/RxJava/wiki/" target="_blank">I assume you already had a taste of it</a>), since <a href="http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/" target="_blank">there are a lot articles</a> and <a href="https://speakerdeck.com/benjchristensen" target="_blank">badasses</a> of this technology that are doing an excellent job out there. However, <strong><span style="color: #333399;">I will point out what makes it interesting in regards of android applications development, and how it has helped me evolve my first approach of clean architecture.</span></strong>
</p>

<p style="text-align: justify;">
  First, I opted for a reactive pattern by converting use cases (called <span style="color: #333399;"><strong>interactors</strong></span> in the clean architecture naming convention) <span style="color: #333399;"><strong>to return Observables<T> which means all the lower layers will follow the chain and return Observables<T> too.</strong></span>
</p>

<pre class="font-size:13 lang:java decode:true" title="UseCase.java">public abstract class UseCase {

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
}</pre>

<p style="text-align: justify;">
  As you can see here, all use cases inherit from this abstract class and implement the abstract method <span style="color: #333399;"><strong>buildUseCaseObservable()</strong></span> which will setup an <span style="color: #333399;"><strong>Observable<T></strong></span> that is going to do the hard job and return the needed data.
</p>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>Something to highlight is the fact that on execute() method, we make sure our Observable<T> executes itself in a separate thread</strong></span>, thus, minimizing how much we block the android main thread. The result is push back on the Android main thread through the android main thread scheduler.
</p>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>So far, we have our Observable<T> up and running, but, as you know, someone has to observe the data sequence emitted by it.</strong></span> To achieve this, I evolved presenters (part of <span style="color: #333399;"><strong>MVP</strong></span> in the presentation layer) into <span style="color: #333399;"><strong>Subscribers</strong></span> which would <span style="color: #333399;"><strong>“react”</strong></span> to these emitted items by use cases, in order to update the user interface.
</p>

<p style="text-align: justify;">
  Here is how the subscriber looks like:
</p>

<pre class="font-size:13 lang:java decode:true" title="UserListPresenter.java">private final class UserListSubscriber extends DefaultSubscriber&lt;List&lt;User&gt;&gt; {

  @Override public void onCompleted() {
    UserListPresenter.this.hideViewLoading();
  }

  @Override public void onError(Throwable e) {
    UserListPresenter.this.hideViewLoading();
    UserListPresenter.this.showErrorMessage(new DefaultErrorBundle((Exception) e));
    UserListPresenter.this.showViewRetry();
  }

  @Override public void onNext(List&lt;User&gt; users) {
    UserListPresenter.this.showUsersCollectionInView(users);
  }
}</pre>

<p style="text-align: justify;">
  Every subscriber is an inner class inside each presenter and implements a <span style="color: #333399;"><strong>DefaultSubscriber<T></strong></span> created basically for default error handling.
</p>

<p style="text-align: justify;">
  After putting all pieces in place, you can get the whole idea by having a look at the following picture:
</p>

[<img class="aligncenter size-full wp-image-360" src="http://fernandocejas.com/wp-content/uploads/2015/07/clean_architecture_evolution.png" alt="clean_architecture_evolution" width="720" height="540" srcset="http://fernandocejas.com/wp-content/uploads/2015/07/clean_architecture_evolution.png 720w, http://fernandocejas.com/wp-content/uploads/2015/07/clean_architecture_evolution-300x225.png 300w" sizes="(max-width: 720px) 100vw, 720px" />](http://fernandocejas.com/wp-content/uploads/2015/07/clean_architecture_evolution.png)

<p style="text-align: justify;">
  Let’s enumerate a bunch of benefits we get out of this RxJava based approach:
</p>

<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Decoupling between Observables and Subscribers:</strong></span> makes maintainability and testing easier.
</li>
<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Simplified asynchronous tasks:</strong></span> java threads and futures are complex to manipulate and synchronize if more than one single level of asynchronous execution is required, so by using schedulers we can jump between background and main thread in an easy way (with no extra effort), especially when we need to update the UI. We also avoid what we call a “callback hell”, which makes our code unreadable and hard to follow up.
</li>
<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Data transformation/composition:</strong></span> we can combine multiple Observables<T>  without affecting the client, which makes our solution more scalable.
</li>
<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Error handling:</strong></span> a signal is emitted to the consumer when an error has occurred within any Observable<T>.
</li>

<p style="text-align: justify;">
  From my point of view there is one drawback, and indeed a price to pay, which has to do with <strong><span style="color: #333399;">the learning curve for developers who are not familiar with the concept.</span></strong> However, you get very valuable stuff out of it. <span style="color: #333399;"><strong>Reactive for the win!</strong></span>
</p>

### Dependency Injection: Dagger 2

<p style="text-align: justify;">
  I’m not going to talk much of dependency injection cause <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">I have already written a whole article</a>, which I strongly recommend you to read, so we can stay on the same page here.
</p>

<p style="text-align: justify;">
  Said that, it is worth mentioning, that by implementing a dependency injection framework like <span style="color: #333399;"><strong>Dagger 2</strong> </span>we gain:
</p>

<li style="text-align: justify;">
  <strong><span style="color: #333399;">Components reuse</span></strong>, since dependencies can be injected and configured externally.
</li>
<li style="text-align: justify;">
  When injecting abstractions as collaborators, we can just change the implementation of any object without having to make a lot of changes in our codebase, since that <span style="color: #333399;"><strong>object instantiation resides in one place isolated and decoupled</strong></span>.
</li>
<li style="text-align: justify;">
  Dependencies can be injected into a component: <strong><span style="color: #333399;">it is possible to inject mock implementations of these dependencies which makes testing easier.</span></strong>
</li>

### Lambda expressions: Retrolambda

<p style="text-align: justify;">
  <strong><span style="color: #333399;">No one will complain about making use of Java 8 lambdas in our code</span></strong>,  and even more when they simplify it and get rid of a lot of boilerplate, as you can see in this piece of code:
</p>

<pre class="font-size:13 lang:java decode:true" title="CloudUserDataStore.java">private final Action1&lt;UserEntity&gt; saveToCacheAction =
    userEntity -&gt; {
      if (userEntity != null) {
        CloudUserDataStore.this.userCache.put(userEntity);
      }
    };</pre>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>However, I have mixed feelings here and will explain why</strong></span>. It turns out that at <a href="https://developers.soundcloud.com/blog/" target="_blank">@SoundCloud</a> we had a discussion around <a href="https://github.com/orfjackal/retrolambda" target="_blank">Retrolambda</a>, <span style="color: #333399;"><strong>mainly whether or not to use it</strong></span> and the outcome was:
</p>

<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Pros:</strong></span> <ul>
    <li>
      Lambdas and method references.
    </li>
    <li>
      Try with resources.
    </li>
    <li>
      Dev karma.
    </li>
  </ul>
</li>

<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Cons:</strong></span> <ul>
    <li style="text-align: justify;">
      Accidental use of Java 8 APIs.
    </li>
    <li style="text-align: justify;">
      3rd part lib, quite intrusive.
    </li>
    <li style="text-align: justify;">
      3rd part gradle plugin to make it work with Android.
    </li>
  </ul>
</li>

<p style="text-align: justify;">
  Finally we decided it was not something that would solve any problems for us: <span style="color: #333399;"><strong>your code looks better and more readable but it was something we could live without, since nowadays all the most powerful IDEs contain code folding options which cover this need, at least in an acceptable manner.</strong></span>
</p>

<p style="text-align: justify;">
  Honestly, the main reason why I used it here, was more to play around it and have a taste of lambdas on Android, although <span style="color: #333399;"><strong>I would probably use it again for a spare time project.</strong></span> I will leave the decision up to you. I am just exposing my field of vision here. <span style="color: #333399;"><strong>Of course the <a href="https://github.com/orfjackal" target="_blank">author</a> of this library deserves my kudos for such an amazing job.</strong></span>
</p>

### Testing approach

<p style="text-align: justify;">
  In terms of testing, not big changes in relation with the first version of the example:
</p>

<ul style="text-align: justify;">
  <li>
    <span style="color: #333399;"><strong>Presentation layer:</strong></span> UI tests with Espresso 2 and Android Instrumentation.
  </li>
  <li>
    <span style="color: #333399;"><strong>Domain layer:</strong></span> JUnit + Mockito since it is a regular Java module.
  </li>
  <li>
    <span style="color: #333399;"><strong>Data layer:</strong></span> Migrated test battery to use Robolectric 3 + JUnit + Mockito. Tests for this layer used to live in a separate Android Module, since back then (at the moment of the first version of the example), <span style="color: #333399;"><strong>there was no built-in unit test support and setting up a framework like robolectric was complicated and required a serie of hacks to make it work properly.</strong></span>
  </li>
</ul>

<p style="text-align: justify;">
  Fortunately that <span style="color: #333399;"><strong>is part of the past</strong> </span>and now everything works out of the box so I could relocated them inside the data module, specifically into its default test location: <span style="color: #333399;"><strong>src/test/java</strong></span> folder.
</p>

### Package organization

<p style="text-align: justify;">
  I consider code/package organization one of the key factors of a good architecture: <span style="color: #333399;"><strong>package structure is the very first thing encountered by a programmer when browsing source code.</strong></span> Everything flows from it. Everything depends on it.
</p>

<p style="text-align: justify;">
  We can distinguish between <span style="color: #333399;"><strong>2 paths</strong></span> you can take to divide up your application into packages:
</p>

<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Package by layer:</strong></span> Each package contains items that usually aren&#8217;t closely related to each other. This results in packages with low cohesion and low modularity, with high coupling between packages. As a result, editing a feature involves editing files across different packages. In addition, deleting a feature can almost never be performed in a single operation.
</li>
<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Package by feature:</strong></span> It uses packages to reflect the feature set. It tries to place all items related to a single feature (and only that feature) into a single package. This results in packages with high cohesion and high modularity, and with minimal coupling between packages. Items that work closely together are placed next to each other. They aren&#8217;t spread out all over the application.
</li>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>My recommendation is to go with packages by features</strong></span>, which bring these main benefits:
</p>

<ul style="text-align: justify;">
  <li>
    <span style="color: #333399;"><strong>Higher Modularity</strong></span>
  </li>
  <li>
    <span style="color: #333399;"><strong>Easier Code Navigation</strong></span>
  </li>
  <li>
    <span style="color: #333399;"><strong>Minimizes Scope</strong></span>
  </li>
</ul>

<p style="text-align: justify;">
  It is also interesting to add that if you are working with <span style="color: #333399;"><strong>feature teams</strong></span> (as we do at <a href="https://twitter.com/soundcloud" target="_blank">@SoundCloud</a>), <span style="color: #333399;"><strong>code ownership will be easier to organize and more modularized</strong></span>, which is a win in a growing organization where many developers work on the same codebase.
</p>

<img class="aligncenter wp-image-368" src="http://fernandocejas.com/wp-content/uploads/2015/07/package_organization-795x1024.png" alt="package_organization" width="450" height="580" srcset="http://fernandocejas.com/wp-content/uploads/2015/07/package_organization-795x1024.png 795w, http://fernandocejas.com/wp-content/uploads/2015/07/package_organization-233x300.png 233w, http://fernandocejas.com/wp-content/uploads/2015/07/package_organization.png 916w" sizes="(max-width: 450px) 100vw, 450px" />

<p style="text-align: justify;">
  As you can see, my approach looks like packages organized by layer: <span style="color: #333399;"><strong>I might have gotten wrong here (and group everything under &#8216;users&#8217; for example)</strong></span> but I will <span style="color: #333399;"><strong>forgive myself in this case,</strong></span> because this sample is for learning purpose and what I wanted to expose, were the main concepts of the clean architecture approach. <span style="color: #333399;"><strong>DO AS I SAY, NOT AS I DO :).</strong></span>
</p>

### Extra ball: organizing your build logic

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>We all know that you build a house from the foundations up.</strong></span> The same happens with software development, and here I want to remark that, from my perspective, <span style="color: #333399;"><strong>the build system (and its organization) is an important piece of a software architecture.</strong></span>
</p>

<p style="text-align: justify;">
  On Android, we use gradle, which is a platform agnostic build system and indeed, very powerful. The idea here is to go through a <span style="color: #333399;"><strong>bunch of tips and tricks</strong></span> that can simplify your life when it comes to how organize the way you build your application:
</p>

<li style="text-align: justify;">
  <span style="color: #333399;"><strong>G</strong><strong><span style="color: #333399;">roup</span> stuff by functionality in separate gradle build files.</strong></span>
</li>

<img class="aligncenter wp-image-372" src="http://fernandocejas.com/wp-content/uploads/2015/07/gradle_organization-283x300.png" alt="gradle_organization" width="210" height="223" srcset="http://fernandocejas.com/wp-content/uploads/2015/07/gradle_organization-283x300.png 283w, http://fernandocejas.com/wp-content/uploads/2015/07/gradle_organization.png 428w" sizes="(max-width: 210px) 100vw, 210px" />

<pre class="font-size:13 lang:java decode:true" title="ci.gradle">def ciServer = 'TRAVIS'
def executingOnCI = "true".equals(System.getenv(ciServer))

// Since for CI we always do full clean builds, we don't want to pre-dex
// See http://tools.android.com/tech-docs/new-build-system/tips
subprojects {
  project.plugins.whenPluginAdded { plugin -&gt;
    if ('com.android.build.gradle.AppPlugin'.equals(plugin.class.name) ||
        'com.android.build.gradle.LibraryPlugin'.equals(plugin.class.name)) {
      project.android.dexOptions.preDexLibraries = !executingOnCI
    }
  }
}</pre>

<pre class="font-size:13 lang:java decode:true" title="build.gradle">apply from: 'buildsystem/ci.gradle'
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
...</pre>

<p style="text-align: justify;">
  Thus, you can use <span style="color: #333399;"><strong>&#8220;apply from: &#8216;buildsystem/ci.gradle’&#8221;</strong></span> to plug that configuration to any gradle build file. <span style="color: #333399;"><strong>Do not put everything on only one build.gradle file otherwise you will start creating a monster. Lesson learned.</strong></span>
</p>

<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Create maps of dependencies</strong></span>
</li>

<pre class="font-size:13 lang:java decode:true" title="dependencies.gradle">...

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
}</pre>

<pre class="font-size:13 lang:java decode:true  " title="build.gradle">apply plugin: 'java'

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
}</pre>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>This is very useful if you wanna reuse the same artifact version across different modules in your project</strong></span>, or maybe the other way around, where you have to apply different dependency versions to different modules. Another plus one, is that <span style="color: #333399;"><strong>you also control the dependencies in one place</strong></span> and, for instance, bumping an artifact version is pretty straightforward.
</p>

### Wrapping up

<p style="text-align: justify;">
  That is pretty much I have for now, and as a conclusion, keep in mind <span style="color: #333399;"><strong>there are no silver bullets</strong></span>. <span style="color: #333399;"><strong>However, a good software architecture will help us keep our code clean and healthy, as well as scalable and easy to maintain.</strong></span>
</p>

<p style="text-align: justify;">
  There is a few more things I would like to point out and they have to do with attitudes you should take when facing a software problem:
</p>

<li style="text-align: justify;">
  <strong><span style="color: #333399;">Respect SOLID principles.</span></strong>
</li>
<li style="text-align: justify;">
  <strong><span style="color: #333399;">Do not over think (do not do over engineering).</span></strong>
</li>
<li style="text-align: justify;">
  <strong><span style="color: #333399;">Be pragmatic.</span></strong>
</li>
<li style="text-align: justify;">
  <strong><span style="color: #333399;">Minimize framework (android) dependencies in your project as much as you can.</span></strong>
</li>

### Source code

  1. <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Clean architecture github repository &#8211; master branch</a>
  2. <a href="https://github.com/android10/Android-CleanArchitecture/releases" target="_blank">Clean architecture github repository &#8211; releases</a>

### Further reading:

  1. <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android..the clean way</a>
  2. <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">Tasting Dagger 2 on Android</a>
  3. <a href="https://speakerdeck.com/android10/the-mayans-lost-guide-to-rxjava-on-android" target="_blank">The Mayans Lost Guide to RxJava on Android</a>
  4. <a href="https://speakerdeck.com/android10/it-is-about-philosophy-culture-of-a-good-programmer" target="_blank">It is about philosophy: Culture of a good programmer</a>

<center>
</center>

<center>
</center>&nbsp;

### References

  1. <a href="https://github.com/ReactiveX/RxJava/wiki" target="_blank">RxJava wiki by Netflix</a>
  2. <a href="https://blog.8thlight.com/uncle-bob/2014/05/11/FrameworkBound.html" target="_blank">Framework bound by Uncle Bob</a>
  3. <a href="https://docs.gradle.org/current/userguide/userguide.html" target="_blank">Gradle user guide</a>
  4. <a href="http://www.javapractices.com/topic/TopicAction.do?Id=205" target="_blank">Package by feature, not layer</a>