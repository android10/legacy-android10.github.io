---
id: 5
title: 'RxJava Observable tranformation: concatMap() vs flatMap()'
date: 2015-01-11T22:58:06+00:00
author: fernando
description: In this article I will explain the main difference between concatmap and flatmap rxjava operators
layout: post
permalink: /2015/01/11/rxjava-observable-tranformation-concatmap-vs-flatmap/
image: assets/images/concat_vs_flatmap_featured.jpg
comments: false
featured: false
hidden: false
categories: [ java, fp, oop, functional, programming, engineering, reactive, rxjava ]
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
After a while I decided that was time to get back for some writing. As you may know at <a href="https://twitter.com/soundcloud" target="_blank">@SoundCloud</a> we do a strong use of the reactive approach, but to be honest, I am not here to talk about <a href="https://github.com/ReactiveX/RxJava" target="_blank">RxJava</a> itself because there are great articles out there to read about it (<a href="https://mttkay.github.io/blog/2013/08/25/functional-reactive-programming-on-android-with-rxjava/" target="_blank">here</a> and <a href="http://techblog.netflix.com/2013/02/rxjava-netflix-api.html" target="_blank">here</a>) and great people to follow as well such as <a href="https://twitter.com/benjchristensen" target="_blank">Ben Christesen</a>, <a href="https://twitter.com/mttkay" target="_blank">Matthias Käppler</a> and <a href="https://www.youtube.com/playlist?list=PLSD48HvrE7-Z1stQ1vIIBumB0wK0s8llY" target="_blank">many others</a>.
  
I also consider myself a "newbie" in reactive programming and now I am at that stage where you start seeing the benefits of this approach and want to make every single object reactive, which is very dangerous, so if you are in the same level as me, just keep an eye on it, and use it wherever makes sense, you are advised.
  
Let"s get started with the article then...


## Observable transformation

There are times where you have an **Observable** which you are subscribed to and you want to transform the results (remember that everything is a stream in **Reactive Programming**).
  
**When it comes to observable transformation, the values from the sequences we consume are not always in the format or shape we need or each value needs to be expanded either into a richer object or into more values,** so we can do this by applying a function to each element returned by your observable which will convert all of the items emitted by it into **Observables** and merge the result. 

Do not worry if you do not understand yet (it took me a while to think in reactive), we will see an example in a bit.


## The problem

**I was retrieving a set of values from the database and applying a function to each of them that was suppose to both transform them in other objects asynchronously and also preserve their order.** 

Last step was to convert them into a list needed by the UI to display the results. 

The behavior I had was not the expected one and here is why: I was using ```Observable.flatMap()``` which does not preserve the order of the elements.


## A simple example

Let me put a simple example to demonstrate the mentioned behavior. 

Let"s say we have an **Observable** emitting a set of Integers and we want to calculate the square of each of those values:

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

Here our **DataManager** class has a method that returns an **Observable** which emits numbers from 2 to 10. 

Then we want to calculate the square of those values so here is our function to apply to each of them:

```java
private final Func1<Integer, Observable<Integer>> SQUARE_OF_NUMBER =
    new Func1<Integer, Observable<Integer>>() {
      @Override public Observable<Integer> call(Integer number) {
        return dataManager.squareOf(number);
      }
    };
```

This will take an **Integer** as entry, will generate an **Observable&lt;Integer&gt;**, merge them and emit the results. 

As you can see we are using a call to ```dataManager.squareOf()``` method which is asynchronous (for demonstration purpose) and looks something like this:

```java
public Observable<Integer> squareOf(int number) {
  return Observable.just(number * number).subscribeOn(Schedulers.from(this.jobExecutor));
}
```

Of course this works, but not as expected (at least the way I wanted), the order of the elements is not preserved (logcat output):

![concat_vs_flatmap](/assets/images/concat_vs_flatmap_01.png)


## Observable flatMap() vs concatMap()

Both methods look pretty much the same, **but there is a difference: operator usage when merging the final results.**

Here is some stuff from the official documentation:

![concat_vs_flatmap](/assets/images/concat_vs_flatmap_02.png)

The ```flatMap()``` method creates a **new Observable** by applying a function that you supply to each item emitted by the original **Observable**, where that function is itself an **Observable** that emits items, and then merges the results of that function applied to every item emitted by the original Observable, emitting these merged results. 

Note that ```flatMap()``` may interleave the items emitted by the Observables that result from transforming the items emitted by the source **Observable**.

**If it is important that these items not be interleaved**, you can instead use the similar ```concatMap()``` method.

![concat_vs_flatmap](/assets/images/concat_vs_flatmap_03.png)

**As you can see, the two functions are very similar and the subtle difference is how the output is created** (after the mapping function is applied . ```flatMap()``` uses **MERGE** operator while ```concatMap()``` uses **CONCAT** operator meaning that the last one cares about the order of the elements, so keep an eye on that if you need ordering :).


## Merge operator

**Combine multiple Observables into one.**
  
![concat_vs_flatmap](/assets/images/concat_vs_flatmap_04.png)


## Concat operator

**Concatenate two or more Observables sequentially.**
  
![concat_vs_flatmap](/assets/images/concat_vs_flatmap_05.png)


## Problem solved

**Observable** ```concatMap()``` for the salvation! The problem was easily solved by just switching to a ```concatMap()``` method. 

I know you may argue why I did not read the documentation first, <a href="https://github.com/ReactiveX/RxJava/wiki" target="_blank">which is very well explained</a> by the way (kudos to the RxJava contributors!!!), but sometimes we are lazy or that is the last place we look into. 

Here is a picture with the final results and some test I did (you can find the sample code below):

![concat_vs_flatmap](/assets/images/concat_vs_flatmap_06.png)


## References

**That is my two cents and hope it helps.**

As always here is the sample code of the sample app and other useful information that worth read.

  * Source code: <a href="https://github.com/android10/Android-ReactiveProgramming" target="_blank">https://github.com/android10/Android-ReactiveProgramming</a>
  * <a href="https://mttkay.github.io/blog/2013/08/25/functional-reactive-programming-on-android-with-rxjava/" target="_blank">Functional Reactive Programming on Android With RxJava</a>
  * <a href="http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/" target="_blank">Grokking RxJava</a>
  * <a href="http://futurice.com/blog/top-7-tips-for-rxjava-on-android" target="_blank">Top 7 Tips for RxJava on Android</a>
  * <a href="http://docs.couchbase.com/developer/java-2.0/observables.html" target="_blank">Mastering Observables</a>
  * <a href="https://www.youtube.com/playlist?list=PLSD48HvrE7-Z1stQ1vIIBumB0wK0s8llY" target="_blank">React Conference London</a>

Remember that any feedback is very welcome, such as better ways of addressing this problem or any issue you may find.