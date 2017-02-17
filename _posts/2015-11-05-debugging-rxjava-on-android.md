---
id: 8
title: Debugging RxJava on Android
date: 2015-11-05T00:12:19+00:00
author: Fernando Cejas
layout: post
permalink: /2015/11/05/debugging-rxjava-on-android/
categories:
  - Android
  - Aspect Oriented Programming
  - Development
  - Java
  - Reactive Programming
tags:
  - android
  - androiddev
  - aop
  - asm
  - asmdex
  - aspect oriented programming
  - aspectj
  - cglib
  - code injection
  - development
  - javassits
  - programming
  - reactive
  - reactive programming
  - weaving
---
<p class="justify"><span class="boldtext">Debugging</span> is the process of finding and resolving bugs or defects that prevent correct operation of computer software (<a href="https://en.wikipedia.org/wiki/Debugging" target="_blank">Wikipedia</a>).</p>

<p class="justify">Nowadays debugging is not an easy task, specially with all the complexity around current systems: <span class="boldtext">Android is not an exception to this rule</span> and since we are dealing with asynchronous executions, that becomes way harder.</p>

<p class="justify">As you might know, at <a href="https://soundcloud.com/" target="_blank">@SoundCloud</a>, we are heavily using <a href="https://github.com/ReactiveX/RxJava" target="_blank">RxJava</a> as one of our core components for <span class="boldtext">Android Development</span>, so in this article I am gonna walk you through the way we debug Rx <a href="http://reactivex.io/documentation/observable.html" target="_blank">Observables</a> and <a href="http://reactivex.io/RxJava/javadoc/rx/Subscriber.html" target="_blank">Subscribers</a>.</p>

## Give a warm welcome to Frodo

<p class="justify">Let me get started by introducing <span class="boldtext">Frodo</span>, but first, if you already watched <a href="https://www.youtube.com/watch?v=R16OHcZJTno" target="_blank">Matthias Käppler talk at GOTO Conference</a> (if you have not yet, I strongly recommend it), you may have noticed that he talks about someone called <a href="https://github.com/android10/gandalf" target="_blank">Gandalf</a> (<a href="https://youtu.be/R16OHcZJTno?t=2475" target="_blank">minute 41:15</a>).</p>

<p class="justify">All right, I have to say that in the beginning, <a href="https://github.com/android10/gandalf" target="_blank">Gandalf was my failed attempt</a> to create an <span class="boldtext">Aspect Oriented Library for Android</span>, fortunately after working hard and receiving useful feedback, it became an <span class="boldtext">Android Development Kit</span> we use at <a href="https://soundcloud.com/" target="_blank">@SoundCloud</a>. However, I wanted to have <span class="boldtext">something smaller that solves only one problem</span>, so I decided to extract <a href="https://github.com/ReactiveX/RxJava" target="_blank">RxJava</a> Logging specifics that I have been working on, and give life to <span class="boldtext">Frodo</span>.</p>

<p class="justify"><span class="boldtext">Frodo</span> is no more than an <span class="boldtext">Android Library for Logging RxJava Observables and Subscribers</span> (for now), let's say <span class="boldtext">Gandalf's</span> little son or brother. It was actually inspired by <a href="https://github.com/JakeWharton/hugo" target="_blank">Jake Wharton's Hugo Library</a>.</p>

<img class="aligncenter wp-image-402 size-full" src="/assets/migrated/frodo_one.jpg" alt="frodo_one" width="450" height="415" srcset="/assets/migrated/frodo_one.jpg 450w, /assets/migrated/frodo_one-300x277.jpg 300w" sizes="(max-width: 450px) 100vw, 450px" />

## Debugging RxJava

<p class="justify">First of all, I assume that you have basic knowledge about <span class="boldtext">RxJava</span> and its core components: <span class="boldtext">Observables</span> and <span class="boldtext">Subscribers</span>.</p>

<p class="justify"><span class="boldtext">Debugging is a cross cutting concern</span> and we know how frustrating and painful could be. Additionally, many times you have to write code (that is not part of your business logic) in order to <span class="boldtext">debug</span> stuff, which make things even more complicated, specially when it comes to <span class="boldtext">asynchronous code execution</span>.</p>

<p class="justify"><span class="boldtext">Frodo was born to achieve this and avoid writing code for debugging RxJava objects. It is based on Java Annotations and relies on a Gradle Plugin</span> that detects when the Debug build type of your application is compiled, and weaves code, which is gonna print <span class="boldtext">RxJava Objects logging information</span> on the android logcat output. For instance, <span class="boldtext">it is safe to keep Frodo annotations in your codebase</span> even when you are generating Release versions of your Android App. So now, let's get our hands dirty and have a taste of it.</p>

## Using Frodo

<p class="justify">To use <span class="boldtext">Frodo</span> the first thing we need to do is to simply apply a <span class="boldtext">Gradle Plugin</span> to our Android Project like this:</p>

```groovy
buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:1.3.1'
    classpath "com.fernandocejas.frodo:frodo-plugin:0.8.1"
  }
}

apply plugin: 'com.android.application'
apply plugin: 'com.fernandocejas.frodo'
```

<p class="justify">As you can see, we add <span class="boldtext">"com.fernandocejas.frodo:frodo-plugin:0.8.1"</span> to the classpath and afterwards we apply the plugin <span class="boldtext">'com.fernandocejas.frodo'</span>. That should be enough to have access to the <span class="boldtext">Java annotations provided by the Library</span>.</p>

## Inspecting @RxLogObservable

<p class="justify"><span class="boldtext">The first core functionality of Frodo is to log RxJava Observables through @RxLogObservable Java annotation</span>. Let's say we have a method that returns an Observable which will emit a list of some sort of DummyClass:</p>

```java
public class ObservableSample {

  @RxLogObservable
  public Observable<List<MyDummyClass>> list() {
    return Observable.just(buildDummyList());
  }
  
  public List<MyDummyClass> buildDummyList() {
    return Arrays.asList(new MyDummyClass("Batman"), new MyDummyClass("Superman"));
  }

  public static final class MyDummyClass {
    private final String name;

    MyDummyClass(String name) {
      this.name = name;
    }

    @Override
    public String toString() {
      return "Name: " + name;
    }
  }
}
```

<p class="justify">Then we subscribe to our sample observable:</p>

```java
final ObservableSample observableSample = new ObservableSample();
observableSample.list()
  .subscribeOn(Schedulers.newThread())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(new Action1<List<ObservableSample.MyDummyClass>>() {
    @Override
    public void call(List<ObservableSample.MyDummyClass> myDummyClasses) {
      toastMessage("onNext() List--> " + myDummyClasses.toString());
    }
  });
```

<p class="justify">When compiling and running our application, this is the information <span class="boldtext">we are gonna see on the android logcat:</span></p>

```
ObservableSample  D  Frodo => [@Observable :: @InClass -> ObservableSample :: @Method -> list()]
                  D  Frodo => [@Observable#list -> onSubscribe() :: @SubscribeOn -> RxNewThreadScheduler-3]
                  D  Frodo => [@Observable#list -> onNext() -> [Name: Batman, Name: Superman]]
                  D  Frodo => [@Observable#list -> onCompleted()]
                  D  Frodo => [@Observable#list -> onTerminate() :: @Emitted -> 1 element :: @Time -> 0 ms]
                  D  Frodo => [@Observable#list -> onUnsubscribe() :: @ObserveOn -> main]
```

<p class="justify"><span class="boldtext">Basically this means that we subscribed to an Observable returned by the list() method in ObservableSample class</span>. Then we get information about the emitted items, schedulers and events triggered by the annotated Observable.</p>

## Inspecting @RxLogSubscriber

<p class="justify"><span class="boldtext">Let's now explore what @RxLogSubscriber is capable of.</span><br /> To put an example, let's create a <span class="boldtext">RxJava</span> dummy Subscriber and annotate it with <span class="boldtext">@RxLogSubscriber.</span></p>

```java
@RxLogSubscriber
public class MySubscriberBackpressure extends Subscriber<Integer> {

  @Override
  public void onStart() {
    request(16);
  }

  @Override
  public void onNext(Integer value) {
    //empty
  }

  @Override
  public void onError(Throwable throwable) {
    //empty
  }

  @Override
  public void onCompleted() {
    if (!isUnsubscribed()) {
      unsubscribe();
    }
  }
}
```

<p class="justify"><span class="boldtext">Forget about the <a href="https://github.com/ReactiveX/RxJava/wiki/Backpressure" target="_blank">backpressure</a> name of this Subscriber for now, since this topic deserves a whole article.</span> Just know that this Subscriber will only request 16 elements and it is gonna do nothing with the items it receives on the <span class="boldtext">onNext()</span> method. Even though that, <span class="boldtext">we still wanna see what is going on when it subscribes to any Observable which emits Integer values</span>:</p>

```java
public class ObservableSample {
  public Observable<Integer> numbersBackpressure() {
    return Observable.create(new Observable.OnSubscribe<Integer>() {
      @Override
      public void call(Subscriber<? super Integer> subscriber) {
        try {
          if (!subscriber.isUnsubscribed()) {
            for (int i = 1; i < 10000; i++) {
              subscriber.onNext(i);
            }
            subscriber.onCompleted();
          }
        } catch (Exception e) {
          subscriber.onError(e);
        }
      }
    });
  }
}
```

<p class="justify">Here is when <span class="boldtext">we subscribe</span> to our SampleObservable:</p>

```java
final ObservableSample observableSample = new ObservableSample();
observableSample.numbersBackpressure()
  .onBackpressureDrop()
  .subscribeOn(Schedulers.newThread())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(new MySubscriberBackpressure());
```

<p class="justify"><span class="boldtext">Again when we compile and run our application,</span> this is what we get from the logcat output:</p>

```
SubscriberBackpressure  D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onStart()]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> @Requested -> 40 elements]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 1 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 2 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 3 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 4 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 5 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 6 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 7 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 8 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 9 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 10 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 11 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 12 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 13 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 14 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 15 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onNext() -> 16 :: @ObserveOn -> main]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> onCompleted() :: @Received -> 16 elements :: @Time -> 3 ms]
                        D  Frodo => [@Subscriber :: MySubscriberBackpressure -> unSubscribe()]
```

<p class="justify"><span class="boldtext">Information here includes each of the items received, number of elements, schedulers, execution time and events triggered.</span></p>

<p class="justify">As you can see this information is useful in cases of <a href="https://github.com/ReactiveX/RxJava/wiki/Backpressure" target="_blank">backpressure</a>, or to see <span class="boldtext">in which thread the items are being emitted</span> or when we wanna se if our <span class="boldtext">Subscriber has subscribed successfully</span>, thus <span class="boldtext">avoiding memory leaks for example.</span></p>

## Frodo under the hood

<p class="justify"><span class="boldtext">In this article, I'm not gonna explain in details how the library internally works,</span> however, if you are curious about it, <a href="http://fernandocejas.com/2014/08/03/aspect-oriented-programming-in-android/" target="_blank">you can check an article I wrote last year</a> which includes an example with the same approach I am using for <span class="boldtext">Frodo</span>.</p>

<p class="justify"><span class="boldtext"><a href="https://speakerdeck.com/android10/android-aspect-oriented-programming" target="_blank">You can also look into a presentation</a></span> I prepared as an introduction for both <a href="https://speakerdeck.com/android10/android-aspect-oriented-programming" target="_blank">AOP and the Library</a> or even better, dive into the source code.</p>

## Disclaimer: Early stage

<p class="justify"><span class="boldtext">Frodo was just born and there is a long way ahead of it. It is still in a very early stage, so you might find issues or things to improve.</span></p>

<p class="justify">
  Actually, one of the main reasons why it was open source, was <span class="boldtext">to receive feedback/input from the community</span> in order to improve it, make it better and more useful. I have to say that I'm very excited and I have already used it in 3 different projects without many problems (check the known issues section below for more information). <span class="boldtext">Of course pull requests are very welcome too.</span>
</p>

## Known issues

<p class="justify"><span class="boldtext">So far, there is a well known issue:</span> since <span class="boldtext">Frodo</span> relies on a <span class="boldtext">Gradle Plugin</span> (as explained earlier) to detect <span class="boldtext">Android Debug build variant and weave code</span>, <span class="boldtext">if you make use of Android Library Projects,</span> when you build your Application (even the debug build type), the <span class="boldtext">official Android Gradle Plugin</span> will always generate release versions of all the Android Library projects included in your solution, thus, <span class="boldtext">this stops Frodo from injecting generated code in annotated methods/classes</span>.</p>

<p class="justify">Of course this is not gonna make your app to crash but you won't see any output on the logcat. <span class="boldtext">There is a workaround </span>for this but be careful if you use it, since you do not wanna ship a release version of your app with business objects being logged all over the place and exposing critical information. Just add this flag to the android section in the <span class="boldtext">build.gradle</span> file of you <span class="boldtext">Android Library Project:</span></p>

```groovy
android {
  defaultPublishConfig "debug"
}
```

## Frodo Example Application

<p class="justify">The <a href="https://github.com/android10/frodo" target="_blank">repository</a> includes a <span class="boldtext">sample app</span> where you can see different use cases, such as <span class="boldtext">Observable errors and other logging information.</span> I have also enabled <span class="boldtext">Frodo</span> in my <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Android Clean Architecture repo</a> if you wanna have a look into it.</p>

## Wrapping up

<p class="justify"><span class="boldtext">This is pretty much I have to offer in this article, and I hope you have found Frodo useful.</span> The first version is out and you can find the repository of the project here: </p>

* <a href="https://github.com/android10/frodo" target="_blank">https://github.com/android10/frodo</a> 

<p class="justify"><span class="boldtext">As always, any feedback is welcome. PRs as well if you wanna contribute.</span> See you soon.</p>

## Useful links

  * <a href="https://github.com/android10/frodo" target="_blank">Frodo Project website.</a>
  * <a href="http://fernandocejas.com/2014/08/03/aspect-oriented-programming-in-android/" target="_blank">Aspect Oriented Programming in Android.</a>
  * <a href="https://speakerdeck.com/android10/android-aspect-oriented-programming" target="_blank">AO Programming and Frodo Presentation.</a>
  * <a href="https://www.youtube.com/watch?v=R16OHcZJTno" target="_blank">Matthias Käppler Talk at GOTO Conference: Going Reactive.</a>