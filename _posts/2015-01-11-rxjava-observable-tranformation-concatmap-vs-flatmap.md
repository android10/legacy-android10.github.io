---
id: 5
title: 'RxJava Observable tranformation: concatMap() vs flatMap()'
date: 2015-01-11T22:58:06+00:00
author: Fernando Cejas
layout: post
permalink: /2015/01/11/rxjava-observable-tranformation-concatmap-vs-flatmap/
categories:
  - Android
  - Development
  - Java
  - Reactive Programming
tags:
  - android
  - androiddev
  - concatMap
  - development
  - flatMap
  - observable
  - observer
  - programming
  - reactive
  - reactive programming
  - rx
  - rxjava
---
<p class="justify"><span class="boldtext">After a while I decided that was time to get back for some writing.</span> As you may know at <a href="https://twitter.com/soundcloud" target="_blank">@SoundCloud</a> we do a strong use of the reactive approach, but to be honest, I am not here to talk about <a href="https://github.com/ReactiveX/RxJava" target="_blank">RxJava</a> itself because there are great articles out there to read about it (<a href="https://mttkay.github.io/blog/2013/08/25/functional-reactive-programming-on-android-with-rxjava/" target="_blank">here</a> and <a href="http://techblog.netflix.com/2013/02/rxjava-netflix-api.html" target="_blank">here</a>) and great people to follow as well such as <a href="https://twitter.com/benjchristensen" target="_blank">Ben Christesen</a>, <a href="https://twitter.com/mttkay" target="_blank">Matthias Käppler</a> and <a href="https://www.youtube.com/playlist?list=PLSD48HvrE7-Z1stQ1vIIBumB0wK0s8llY" target="_blank">many others</a>.</p>
  
<p class="justify">I also consider myself a <span class="boldtext">"newbie"</span> in reactive programming and now I am at that stage where you start seeing the benefits of this approach and want to make every single object reactive, which is very dangerous, <span class="boldtext">so if you are in the same level as me, just keep an eye on it, and use it wherever makes sense, you are advised.</span></p>
  
<p class="justify">Let"s get started with the article then...</p>

## Observable transformation

<p class="justify">There are times where you have an <span class="boldtext">Observable</span> which you are subscribed to and you want to transform the results (remember that everything is a stream in Reactive Programming).</p>
  
<p class="justify"><span class="boldtext">When it comes to observable transformation, the values from the sequences we consume are not always in the format or shape we need or each value needs to be expanded either into a richer object or into more values,</span> so we can do this by applying a function to each element returned by your observable which will convert all of the items emitted by it into <span class="boldtext">Observables</span> and merge the result. Do not worry if you do not understand yet (it took me a while to think in reactive), we will see an example in a bit.</p>

## The problem

<p class="justify"><span class="boldtext">I was retrieving a set of values from the database and applying a function to each of them that was suppose to both transform them in other objects asynchronously and also preserve their order.</span> Last step was to convert them into a list needed by the UI to display the results. The behavior I had was not the expected one and here is why: I was using <span class="boldtext">Observable.flatMap() which does not preserve the order of the elements.</span></p>

## A simple example

<p class="justify"><span class="boldtext">Let me put a simple example to demonstrate the mentioned behavior.</span> Let"s say we have an <span class="boldtext">Observable</span> emitting a set of Integers and we want to calculate the square of each of those values:</p>

```java
public class DataManager {
  private final List<Integer> numbers;
  private final Executor jobExecutor;

  public DataManager() {
    this.numbers = new ArrayList<>(Arrays.asList(2, 3, 4, 5, 6, 7, 8, 9, 10));
    jobExecutor = JobExecutor.getInstance();
  }

  public Observable<Integer> getNumbers() {
    return Observable.from(numbers);
  }

  public List<Integer> getNumbersSync() {
    return this.numbers;
  }

  public Observable<Integer> squareOf(int number) {
    return Observable.just(number * number).subscribeOn(Schedulers.from(this.jobExecutor));
  }
}
```

<p class="justify">Here our <span class="boldtext">DataManager</span> class has a method that returns an <span class="boldtext">Observable</span> which emits numbers from 2 to 10. Then we want to calculate the square of those values so here is our <span class="boldtext">function</span> to apply to each of them:</p>

```java
private final Func1<Integer, Observable<Integer>> SQUARE_OF_NUMBER =
    new Func1<Integer, Observable<Integer>>() {
      @Override public Observable<Integer> call(Integer number) {
        return dataManager.squareOf(number);
      }
    };
```

<p class="justify">This will take an <span class="boldtext">Integer</span> as entry, will generate an <span class="boldtext">Observable&lt;Integer&gt;,</span> merge them and emit the results. As you can see we are using a call to <span class="boldtext">dataManager.squareOf()</span> method which is asynchronous (for demonstration purpose) and looks something like this:</p>

```java
public Observable<Integer> squareOf(int number) {
  return Observable.just(number * number).subscribeOn(Schedulers.from(this.jobExecutor));
}
```

<p class="justify">Of course this works, but not as expected (at least the way I wanted), <span class="boldtext">the order of the elements is not preserved (logcat output):</span></p>

<img class="aligncenter wp-image-255 size-medium" src="/assets/migrated/flatMap_logcat-300x154.png" alt="flatMap_logcat" width="300" height="154" srcset="/assets/migrated/flatMap_logcat-300x154.png 300w, /assets/migrated/flatMap_logcat.png 590w" sizes="(max-width: 300px) 100vw, 300px" />

## Observable flatMap() vs concatMap()

<p class="justify"><span class="boldtext">Both methods look pretty much the same, but there is a difference: operator usage when merging the final results.</span> Here is some stuff from the official documentation:</p>

<img class="aligncenter wp-image-249 size-large" src="/assets/migrated/flatMap-1024x496.png" alt="flatMap" width="640" height="310" srcset="/assets/migrated/flatMap-1024x496.png 1024w, /assets/migrated/flatMap-300x145.png 300w, /assets/migrated/flatMap.png 1280w" sizes="(max-width: 640px) 100vw, 640px" />

<p class="justify">The <span class="boldtext">flatMap()</span> method creates a new <span class="boldtext">Observable</span> by applying a function that you supply to each item emitted by the original <span class="boldtext">Observable,</span> where that function is itself an Observable that emits items, and then merges the results of that function applied to every item emitted by the original Observable, emitting these merged results. Note that <span class="boldtext">flatMap()</span> may interleave the items emitted by the Observables that result from transforming the items emitted by the source Observable. If it is important that these items not be interleaved, you can instead use the similar <span class="boldtext">concatMap()</span> method.</p>

<img class="aligncenter wp-image-248 size-large" src="/assets/migrated/concatMap-1024x488.png" alt="concatMap" width="640" height="305" srcset="/assets/migrated/concatMap-1024x488.png 1024w, /assets/migrated/concatMap-300x143.png 300w, /assets/migrated/concatMap.png 1280w" sizes="(max-width: 640px) 100vw, 640px" />

<p class="justify">As you can see, the two functions are very similar and the subtle difference is how the output is created (after the mapping function is applied). <span class="boldtext">flatMap() uses merge operator while concatMap() uses concat operator meaning that the last one cares about the order of the elements,</span> so keep an eye on that if you need ordering :).</p>

## Merge operator

<p class="justify">Combine multiple <span class="boldtext">Observables</span> into one.</p>
  
[<img class="aligncenter wp-image-256 size-large" src="/assets/migrated/merge-1024x608.png" alt="merge" width="640" height="380" srcset="/assets/migrated/merge-1024x608.png 1024w, /assets/migrated/merge-300x178.png 300w, /assets/migrated/merge.png 1280w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/migrated/merge.png)

## Concat operator

<p class="justify">Concatenate two or more <span class="boldtext">Observables</span> sequentially.</p>
  
[<img class="aligncenter wp-image-253 size-large" src="/assets/migrated/concat-1024x608.png" alt="concat" width="640" height="380" srcset="/assets/migrated/concat-1024x608.png 1024w, /assets/migrated/concat-300x178.png 300w, /assets/migrated/concat.png 1280w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/migrated/concat.png)

## Problem solved

<p class="justify"><span class="boldtext">Observable concatMap() for the salvation! The problem was easily solved by just switching to a concatMap() method.</span> I know you may argue why I did not read the documentation first, <a href="https://github.com/ReactiveX/RxJava/wiki" target="_blank">which is very well explained</a> by the way <span class="boldtext">(kudos to the RxJava contributors!!!),</span> but sometimes we are lazy or that is the last place we look into. Here is a picture with the final results and some test I did (you can find the sample code below):</p>

[<img class="aligncenter wp-image-254 " src="/assets/migrated/final_results_concatMap-576x1024.png" alt="final_results_concatMap" width="348" height="619" srcset="/assets/migrated/final_results_concatMap-576x1024.png 576w, /assets/migrated/final_results_concatMap-169x300.png 169w, /assets/migrated/final_results_concatMap.png 1080w" sizes="(max-width: 348px) 100vw, 348px" />](/assets/migrated/final_results_concatMap.png)

## References

<p class="justify"><span class="boldtext">That is my two cents and hope it helps.</span> As always here is the sample code of the sample app and other useful information that worth read.</p>

  * <span class="boldtext">Source code:</span> <a href="https://github.com/android10/Android-ReactiveProgramming" target="_blank">https://github.com/android10/Android-ReactiveProgramming</a>
  * <a href="https://mttkay.github.io/blog/2013/08/25/functional-reactive-programming-on-android-with-rxjava/" target="_blank">Functional Reactive Programming on Android With RxJava</a>
  * <a href="http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/" target="_blank">Grokking RxJava</a>
  * <a href="http://futurice.com/blog/top-7-tips-for-rxjava-on-android" target="_blank">Top 7 Tips for RxJava on Android</a>
  * <a href="http://docs.couchbase.com/developer/java-2.0/observables.html" target="_blank">Mastering Observables</a>
  * <a href="https://www.youtube.com/playlist?list=PLSD48HvrE7-Z1stQ1vIIBumB0wK0s8llY" target="_blank">React Conference London</a>

<p class="justify"><span class="boldtext">Remember that any feedback is very welcome,</span> such as better ways of addressing this problem or any issue you may find.</p>