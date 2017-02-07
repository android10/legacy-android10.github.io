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
<p style="text-align: justify;">
  <span style="color: #000080;"><strong>Debugging</strong></span> is the process of finding and resolving bugs or defects that prevent correct operation of computer software (<a href="https://en.wikipedia.org/wiki/Debugging" target="_blank">Wikipedia</a>).
</p>

<p style="text-align: justify;">
  Nowadays debugging is not an easy task, specially with all the complexity around current systems: <strong><span style="color: #000080;">Android is not an exception to this rule</span></strong> and since we are dealing with asynchronous executions, that becomes way harder.
</p>

<p style="text-align: justify;">
  As you might know, at <a href="https://soundcloud.com/" target="_blank">@SoundCloud</a>, we are heavily using <a href="https://github.com/ReactiveX/RxJava" target="_blank">RxJava</a> as one of our core components for <strong><span style="color: #000080;">Android Development</span></strong>, so in this article I am gonna walk you through the way we debug Rx <a href="http://reactivex.io/documentation/observable.html" target="_blank">Observables</a> and <a href="http://reactivex.io/RxJava/javadoc/rx/Subscriber.html" target="_blank">Subscribers</a>.
</p>

### Give a warm welcome to Frodo

<p style="text-align: justify;">
  Let me get started by introducing <strong><span style="color: #000080;">Frodo</span></strong>, but first, if you already watched <a href="https://www.youtube.com/watch?v=R16OHcZJTno" target="_blank">Matthias Käppler talk at GOTO Conference</a> (if you haven&#8217;t yet, I strongly recommend it), you may have noticed that he talks about someone called <a href="https://github.com/android10/gandalf" target="_blank">Gandalf</a> (<a href="https://youtu.be/R16OHcZJTno?t=2475" target="_blank">minute 41:15</a>). All right, I have to say that in the beginning, <a href="https://github.com/android10/gandalf" target="_blank">Gandalf was my failed attempt</a> to create an <strong><span style="color: #000080;">Aspect Oriented Library for Android</span></strong>, fortunately after working hard and receiving useful feedback, it became an <strong><span style="color: #000080;">Android Development Kit</span></strong> we use at <a href="https://soundcloud.com/" target="_blank">@SoundCloud</a>. However, I wanted to have <strong><span style="color: #000080;">something smaller that solves only one problem</span></strong>, so I decided to extract <a href="https://github.com/ReactiveX/RxJava" target="_blank">RxJava</a> Logging specifics that I have been working on, and give life to <strong><span style="color: #000080;">Frodo</span></strong>.
</p>

<p style="text-align: justify;">
  <strong><span style="color: #000080;">Frodo</span></strong> is no more than an <strong><span style="color: #000080;">Android Library for Logging RxJava Observables and Subscribers</span></strong> (for now), let&#8217;s say <span style="color: #000080;"><strong>Gandalf&#8217;s</strong></span> little son or brother. It was actually inspired by <a href="https://github.com/JakeWharton/hugo" target="_blank">Jake Wharton&#8217;s Hugo Library</a>.
</p>

<img class="aligncenter wp-image-402 size-full" src="http://fernandocejas.com/wp-content/uploads/2015/11/frodo_one.jpg" alt="frodo_one" width="450" height="415" srcset="http://fernandocejas.com/wp-content/uploads/2015/11/frodo_one.jpg 450w, http://fernandocejas.com/wp-content/uploads/2015/11/frodo_one-300x277.jpg 300w" sizes="(max-width: 450px) 100vw, 450px" />

### Debugging RxJava

<p style="text-align: justify;">
  First of all, I assume that you have basic knowledge about <span style="color: #000080;"><strong>RxJava</strong></span> and its core components: <strong><span style="color: #000080;">Observables</span></strong> and <span style="color: #000080;"><strong>Subscribers</strong></span>.
</p>

<p style="text-align: justify;">
  <strong><span style="color: #000080;">Debugging is a cross cutting concern</span></strong> and we know how frustrating and painful could be. Additionally, many times you have to write code (that is not part of your business logic) in order to <strong><span style="color: #000080;">debug</span></strong> stuff, which make things even more complicated, specially when it comes to <strong><span style="color: #000080;">asynchronous code execution</span></strong>.
</p>

<p style="text-align: justify;">
  <span style="color: #000080;"><strong>Frodo was born to achieve this and avoid writing code for debugging RxJava objects. It is based on Java Annotations and relies on a Gradle Plugin</strong></span> that detects when the Debug build type of your application is compiled, and weaves code, which is gonna print <strong><span style="color: #000080;">RxJava Objects logging information</span></strong> on the android logcat output. For instance, <strong><span style="color: #000080;">it is safe to keep Frodo annotations in your codebase</span></strong> even when you are generating Release versions of your Android App. So now, let&#8217;s get our hands dirty and have a taste of it.
</p>

### Using Frodo

<p style="text-align: justify;">
  To use <strong><span style="color: #000080;">Frodo</span></strong> the first thing we need to do is to simply apply a <strong><span style="color: #000080;">Gradle Plugin</span></strong> to our Android Project like this:
</p>

<pre class="font-size:13 lang:java decode:true" title="build.gradle">buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:1.3.1'
    classpath "com.fernandocejas.frodo:frodo-plugin:0.8.1"
  }
}

apply plugin: 'com.android.application'
apply plugin: 'com.fernandocejas.frodo'</pre>

<p style="text-align: justify;">
  As you can see, we add <span style="color: #000080;"><strong>&#8220;com.fernandocejas.frodo:frodo-plugin:0.8.1&#8221;</strong></span> to the classpath and afterwards we apply the plugin <span style="color: #000080;"><strong>&#8216;com.fernandocejas.frodo&#8217;</strong></span>.<br /> That should be enough to have access to the <span style="color: #000080;"><strong>Java annotations provided by the Library</strong></span>.
</p>

### Inspecting @RxLogObservable

<p style="text-align: justify;">
  <strong><span style="color: #000080;">The first core functionality of Frodo is to log RxJava Observables through @RxLogObservable Java annotation</span></strong>. Let&#8217;s say we have a method that returns an Observable which will emit a list of some sort of DummyClass:
</p>

<pre class="font-size:13 lang:java decode:true" title="ObservableSample.java">public class ObservableSample {

  @RxLogObservable
  public Observable&lt;List&lt;MyDummyClass&gt;&gt; list() {
    return Observable.just(buildDummyList());
  }
  
  public List&lt;MyDummyClass&gt; buildDummyList() {
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
}</pre>

<p style="text-align: justify;">
  Then we subscribe to our sample observable:
</p>

<pre class="font-size:13 lang:java decode:true" title="MyClass.java">final ObservableSample observableSample = new ObservableSample();
observableSample.list()
  .subscribeOn(Schedulers.newThread())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(new Action1&lt;List&lt;ObservableSample.MyDummyClass&gt;&gt;() {
    @Override
    public void call(List&lt;ObservableSample.MyDummyClass&gt; myDummyClasses) {
      toastMessage("onNext() List--&gt; " + myDummyClasses.toString());
    }
  });</pre>

<p style="text-align: justify;">
  When compiling and running our application, this is the information we are gonna see on the logcat:
</p>

<pre class="font-size:11 lang:sh decode:true" title="Logcat">ObservableSample  D  Frodo =&gt; [@Observable :: @InClass -&gt; ObservableSample :: @Method -&gt; list()]
                        D  Frodo =&gt; [@Observable#list -&gt; onSubscribe() :: @SubscribeOn -&gt; RxNewThreadScheduler-3]
                        D  Frodo =&gt; [@Observable#list -&gt; onNext() -&gt; [Name: Batman, Name: Superman]]
                        D  Frodo =&gt; [@Observable#list -&gt; onCompleted()]
                        D  Frodo =&gt; [@Observable#list -&gt; onTerminate() :: @Emitted -&gt; 1 element :: @Time -&gt; 0 ms]
                        D  Frodo =&gt; [@Observable#list -&gt; onUnsubscribe() :: @ObserveOn -&gt; main]</pre>

<p style="text-align: justify;">
  <strong><span style="color: #000080;">Basically this means that we subscribed to an Observable returned by the list() method in ObservableSample class</span></strong>. Then we get information about the emitted items, schedulers and events triggered by the annotated Observable.
</p>

### Inspecting @RxLogSubscriber

<p style="text-align: justify;">
  <strong><span style="color: #000080;">Let&#8217;s now explore what @RxLogSubscriber is capable of.</span></strong><br /> To put an example, let&#8217;s create a <span style="color: #000080;"><strong>RxJava</strong></span> dummy Subscriber and annotate it with <span style="color: #000080;"><strong>@RxLogSubscriber.</strong></span>
</p>

<pre class="font-size:13 lang:java decode:true" title="MySubscriberBackpressure.java">@RxLogSubscriber
public class MySubscriberBackpressure extends Subscriber&lt;Integer&gt; {

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
}</pre>

<p style="text-align: justify;">
  Forget about the <a href="https://github.com/ReactiveX/RxJava/wiki/Backpressure" target="_blank">backpressure</a> name of this Subscriber for now, since this topic deserves a whole article. Just know that this Subscriber will only request 16 elements and it is gonna do nothing with the items it receives on the <strong><span style="color: #000080;">onNext()</span></strong> method. Even though that, <strong><span style="color: #000080;">we still wanna see what is going on when it subscribes to any Observable which emits Integer values</span></strong>:
</p>

<pre class="font-size:13 lang:java decode:true" title="ObservableSample.java">public class ObservableSample {

  public Observable&lt;Integer&gt; numbersBackpressure() {
    return Observable.create(new Observable.OnSubscribe&lt;Integer&gt;() {
      @Override
      public void call(Subscriber&lt;? super Integer&gt; subscriber) {
        try {
          if (!subscriber.isUnsubscribed()) {
            for (int i = 1; i &lt; 10000; i++) {
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
}</pre>

<p style="text-align: justify;">
  Here is when we subscribe to our SampleObservable:
</p>

<pre class="font-size:13 lang:java decode:true" title="MyClass.java">final ObservableSample observableSample = new ObservableSample();
observableSample.numbersBackpressure()
  .onBackpressureDrop()
  .subscribeOn(Schedulers.newThread())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(new MySubscriberBackpressure());</pre>

<p style="text-align: justify;">
  Again when we compile and run our application, this is what we get from the logcat output:
</p>

<pre class="font-size:11 lang:sh decode:true " title="Logcat">SubscriberBackpressure  D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onStart()]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; @Requested -&gt; 40 elements]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 1 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 2 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 3 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 4 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 5 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 6 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 7 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 8 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 9 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 10 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 11 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 12 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 13 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 14 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 15 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onNext() -&gt; 16 :: @ObserveOn -&gt; main]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; onCompleted() :: @Received -&gt; 16 elements :: @Time -&gt; 3 ms]
                        D  Frodo =&gt; [@Subscriber :: MySubscriberBackpressure -&gt; unSubscribe()]</pre>

<p style="text-align: justify;">
  Information here includes each of the items received, number of elements, schedulers, execution time and events triggered.
</p>

<p style="text-align: justify;">
  As you can see this information is useful in cases of <a href="https://github.com/ReactiveX/RxJava/wiki/Backpressure" target="_blank">backpressure</a>, or to see <strong><span style="color: #000080;">in which thread the items are being emitted</span></strong> or when we wanna se if our <strong><span style="color: #000080;">Subscriber has subscribed successfully</span></strong>, thus <strong><span style="color: #000080;">avoiding memory leaks for example.</span></strong>
</p>

### Frodo under the hood

<p style="text-align: justify;">
  In this article, I&#8217;m not gonna explain in details how the library internally works, however, if you are curious about it, <a href="http://fernandocejas.com/2014/08/03/aspect-oriented-programming-in-android/" target="_blank">you can check an article I wrote last year</a> which includes an example with the same approach I am using for <span style="color: #000080;"><strong>Frodo</strong></span>.
</p>

<p style="text-align: justify;">
  <a href="https://speakerdeck.com/android10/android-aspect-oriented-programming" target="_blank">You can also look into a presentation</a> I prepared as an introduction for both <a href="https://speakerdeck.com/android10/android-aspect-oriented-programming" target="_blank">AOP and the Library</a> or even better, dive into the source code.
</p>

### Disclaimer: Early stage

<p style="text-align: justify;">
  <span style="color: #000080;"><strong>Frodo was just born and there is a long way ahead of it. It is still in a very early stage, so you might find issues or things to improve.</strong></span>
</p>

<p style="text-align: justify;">
  Actually, one of the main reasons why it was open source, was <strong><span style="color: #000080;">to receive feedback/input from the community</span></strong> in order to improve it, make it better and more useful. I have to say that I&#8217;m very excited and I have already used it in 3 different projects without many problems (check the known issues section below for more information). <strong><span style="color: #000080;">Of course pull requests are very welcome too.</span></strong>
</p>

### Known issues

<p style="text-align: justify;">
  <span style="color: #000080;"><strong>So far, there is a well known issue:</strong></span> since <span style="color: #000080;"><strong>Frodo</strong></span> relies on a <strong><span style="color: #000080;">Gradle Plugin</span></strong> (as explained earlier) to detect <span style="color: #000080;"><strong>Android Debug build variant and weave code</strong></span>, <span style="color: #000080;"><strong>if you make use of Android Library Projects,</strong></span> when you build your Application (even the debug build type), the <span style="color: #000080;"><strong>official Android Gradle Plugin</strong></span> will always generate release versions of all the Android Library projects included in your solution, thus, <span style="color: #000080;"><strong>this stops Frodo from injecting generated code in annotated methods/classes</strong></span>. Of course this is not gonna make your app to crash but you won&#8217;t see any output on the logcat. <span style="color: #000080;"><strong>There is a workaround</strong> </span>for this but be careful if you use it, since you do not wanna ship a release version of your app with business objects being logged all over the place and exposing critical information.<br /> Just add this flag to the android section in the <strong><span style="color: #000080;">build.gradle</span></strong> file of you <strong><span style="color: #000080;">Android Library Project:</span></strong>
</p>

<pre class="font-size:13 lang:java decode:true" title="build.gradle">android {
  defaultPublishConfig "debug"
}</pre>

### Frodo Example Application

<p style="text-align: justify;">
  The <a href="https://github.com/android10/frodo" target="_blank">repository</a> includes a <span style="color: #000080;"><strong>sample app</strong></span> where you can see different use cases, such as <strong><span style="color: #000080;">Observable errors and other logging information.</span></strong> I have also enabled <span style="color: #000080;"><strong>Frodo</strong></span> in my <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Android Clean Architecture repo</a> if you wanna have a look into it.
</p>

### Wrapping up

<p style="text-align: justify;">
  <span style="color: #000080;"><strong>This is pretty much I have to offer in this article, and I hope you have found Frodo useful.</strong></span><br /> The first version is out and you can find the repository of the project here:<br /> <a href="https://github.com/android10/frodo" target="_blank">https://github.com/android10/frodo</a><br /> As always, any feedback is welcome. <span style="color: #000080;"><strong>PRs as well if you wanna contribute.</strong></span> See you soon.
</p>

### Useful links

  * <a href="https://github.com/android10/frodo" target="_blank">Frodo Project website.</a>
  * <a href="http://fernandocejas.com/2014/08/03/aspect-oriented-programming-in-android/" target="_blank">Aspect Oriented Programming in Android.</a>
  * <a href="https://speakerdeck.com/android10/android-aspect-oriented-programming" target="_blank">AO Programming and Frodo Presentation.</a>
  * <a href="https://www.youtube.com/watch?v=R16OHcZJTno" target="_blank">Matthias Käppler Talk at GOTO Conference: Going Reactive.</a>