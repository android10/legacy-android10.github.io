---
id: 8
title: Debugging RxJava on Android
date: 2015-11-05T00:12:19+00:00
author: fernando
description: Debugging RxJava on Android by using an open source library called frodo
layout: post
permalink: /2015/11/05/debugging-rxjava-on-android/
image: assets/images/debug_rxjava_android_featured.jpg
comments: false
featured: false
hidden: false
categories: [ android, mobile, java, architecture, aop, oop, programming, reactive, rxjava ]
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
Debugging is the process of finding and resolving bugs or defects that prevent correct operation of computer software (<a href="https://en.wikipedia.org/wiki/Debugging" target="_blank">Wikipedia</a>).

**Nowadays debugging is not an easy task, specially with all the complexity around current systems:** Android is not an exception to this rule and since we are dealing with asynchronous executions, that makes the process even harder.

As you might know, at <a href="https://soundcloud.com/" target="_blank">@SoundCloud</a>, we are heavily using <a href="https://github.com/ReactiveX/RxJava" target="_blank">RxJava</a> as one of our core components for Android Development, so in this article I am gonna walk you through the way we debug Rx <a href="http://reactivex.io/documentation/observable.html" target="_blank">Observables</a> and <a href="http://reactivex.io/RxJava/javadoc/rx/Subscriber.html" target="_blank">Subscribers</a>.


## Give a warm welcome to Frodo

Let me get started by introducing **Frodo**, but first, if you already watched <a href="https://www.youtube.com/watch?v=R16OHcZJTno" target="_blank">Matthias Käppler talk at GOTO Conference</a> (if you have not yet, I strongly recommend it), you may have noticed that he talks about someone called <a href="https://github.com/android10/gandalf" target="_blank">Gandalf</a> (<a href="https://youtu.be/R16OHcZJTno?t=2475" target="_blank">minute 41:15</a>).

All right, I have to say that in the beginning, <a href="https://github.com/android10/gandalf" target="_blank">Gandalf was my failed attempt</a> to create an **Aspect Oriented Library for Android**, fortunately after working hard and receiving useful feedback, it became an **Android Development Kit** we use at <a href="https://soundcloud.com/" target="_blank">@SoundCloud</a>. 

However, I wanted to have something smaller that solves only one problem, so I decided to extract <a href="https://github.com/ReactiveX/RxJava" target="_blank">RxJava</a> Logging specifics that I have been working on, and **give life to Frodo.**

**Frodo** is no more than **an Android Library for Logging RxJava Observables and Subscribers** (for now), let's say Gandalf's little son or brother. It was actually inspired by <a href="https://github.com/JakeWharton/hugo" target="_blank">Jake Wharton's Hugo Library</a>.

![debug_rxjava_android](/assets/images/debug_rxjava_android_01.jpg)


## Debugging RxJava

First of all, I assume that you have basic knowledge about **RxJava** and its core components: **Observables** and **Subscribers**.

**Debugging is a cross cutting concern and we know how frustrating and painful could be**. Additionally, many times you have to write code (that is not part of your business logic) in order to debug stuff, which makes things even more complicated, specially when it comes to asynchronous code execution.

**Frodo was born to achieve this and avoid writing code for debugging RxJava objects**. It is based on **Java Annotations** and relies on a **Gradle Plugin** that detects when the Debug build type of your application is compiled, and weaves code, **which is gonna print RxJava Objects logging information on the android logcat output**. 

**For instance, it is safe to keep Frodo annotations in your codebase** even when you are generating Release versions of your Android App. 

So now, let's get our hands dirty and have a taste of it.


## Using Frodo

To use Frodo the first thing we need to do is to simply apply a Gradle Plugin to our Android Project like this:

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

As you can see, we add ```com.fernandocejas.frodo:frodo-plugin:0.8.1``` to the classpath and afterwards we apply the plugin ```com.fernandocejas.frodo```. That should be enough to have access to the Java annotations provided by the Library.


## Inspecting @RxLogObservable

The first core functionality of Frodo is to log RxJava Observables through ```@RxLogObservable``` Java annotation. 

Let's say we have a method that returns an **Observable** which will emit a list of some sort of **DummyClass**:

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

Then we subscribe to our sample observable:

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

When compiling and running our application, this is the information we are gonna see on the android logcat:

```
ObservableSample  D  Frodo => [@Observable :: @InClass -> ObservableSample :: @Method -> list()]
                  D  Frodo => [@Observable#list -> onSubscribe() :: @SubscribeOn -> RxNewThreadScheduler-3]
                  D  Frodo => [@Observable#list -> onNext() -> [Name: Batman, Name: Superman]]
                  D  Frodo => [@Observable#list -> onCompleted()]
                  D  Frodo => [@Observable#list -> onTerminate() :: @Emitted -> 1 element :: @Time -> 0 ms]
                  D  Frodo => [@Observable#list -> onUnsubscribe() :: @ObserveOn -> main]
```

Basically this means that we subscribed to an **Observable** returned by the ```list()``` method in **ObservableSample class**. 

Then we get information about the emitted items, schedulers and events triggered by the annotated Observable.


## Inspecting @RxLogSubscriber

Let's now explore what ```@RxLogSubscriber`` is capable of. 

To put an example, let's create a RxJava dummy Subscriber and annotate it with ```@RxLogSubscriber```.

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

Forget about the <a href="https://github.com/ReactiveX/RxJava/wiki/Backpressure" target="_blank">backpressure</a> name of this Subscriber for now, since this topic deserves a whole article. Just know that **this Subscriber will only request 16 elements** and it is gonna do nothing with the items it receives on the ```onNext()``` method. 

Even though that, we still wanna see what is going on when it subscribes to any **Observable** which emits **Integer** values:

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

Here is when we subscribe to our **SampleObservable**:

```java
final ObservableSample observableSample = new ObservableSample();
observableSample.numbersBackpressure()
  .onBackpressureDrop()
  .subscribeOn(Schedulers.newThread())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(new MySubscriberBackpressure());
```

Again when we compile and run our application, this is what we get from the logcat output:

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

Information here includes each of the **items received**, **number of elements**, **schedulers**, **execution time** and **events triggered**.

As you can see this information is useful in cases of <a href="https://github.com/ReactiveX/RxJava/wiki/Backpressure" target="_blank">backpressure</a> or to see in which thread the items are being emitted or when we wanna se if our Subscriber has subscribed successfully, thus avoiding memory leaks for example.


## Frodo under the hood

In this article, I'm not gonna explain in details how the library internally works, however, if you are curious about it, <a href="http://fernandocejas.com/2014/08/03/aspect-oriented-programming-in-android/" target="_blank">you can check an article I wrote last year</a> which includes an example with the same approach I am using for Frodo.

<a href="https://speakerdeck.com/android10/android-aspect-oriented-programming" target="_blank">You can also look into a presentation</a> I prepared as an introduction for both <a href="https://speakerdeck.com/android10/android-aspect-oriented-programming" target="_blank">AOP and the Library</a> or even better, dive into the source code.


## Disclaimer: Early stage

**Frodo was just born and there is a long way ahead of it. It is still in a very early stage, so you might find issues or things to improve.**

Actually, one of the main reasons why it was open source, was to receive feedback/input from the community in order to improve it, make it better and more useful. 

I have to say that I'm very excited and I have already used it in 3 different projects without many problems (check the known issues section below for more information). 

**Of course pull requests are very welcome too.**


## Known issues

So far, there is a well known issue: since Frodo relies on a Gradle Plugin (as explained earlier) to detect Android Debug build variant and weave code, **if you make use of Android Library Projects, when you build your Application (even the debug build type), the official Android Gradle Plugin will always generate release versions of all the Android Library projects included in your solution**, thus, this stops Frodo from injecting generated code in annotated methods/classes.

Of course this is not gonna make your app to crash but you won't see any output on the logcat. There is a workaround for this but be careful if you use it, since you do not wanna ship a release version of your app with business objects being logged all over the place and exposing critical information. 

Just add this flag to the android section in the ```build.gradle``` file of you **Android Library Project**:

```groovy
android {
  defaultPublishConfig "debug"
}
```


## Frodo Example Application

The <a href="https://github.com/android10/frodo" target="_blank">repository</a> includes a sample app where you can see different use cases, such as **Observable errors and other logging information**. 

I have also enabled Frodo in my <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Android Clean Architecture repo</a> if you wanna have a look into it.


## Wrapping up

This is pretty much I have to offer in this article, and **I hope you have found Frodo useful.** 

The first version is out and you can find the repository of the project here: 

* <a href="https://github.com/android10/frodo" target="_blank">https://github.com/android10/frodo</a> 

As always, any feedback is welcome. PRs as well if you wanna contribute. See you soon.


## Useful links

  * <a href="https://github.com/android10/frodo" target="_blank">Frodo Project website.</a>
  * <a href="http://fernandocejas.com/2014/08/03/aspect-oriented-programming-in-android/" target="_blank">Aspect Oriented Programming in Android.</a>
  * <a href="https://speakerdeck.com/android10/android-aspect-oriented-programming" target="_blank">AO Programming and Frodo Presentation.</a>
  * <a href="https://www.youtube.com/watch?v=R16OHcZJTno" target="_blank">Matthias Käppler Talk at GOTO Conference: Going Reactive.</a>