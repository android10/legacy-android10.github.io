---
id: 9
title: How to use Optional values on Java and Android
date: 2016-02-20T23:19:45+00:00
author: fernando
description: Optional are not available in Android so in this article I will explain how we can make use of them by introducing a library called arrow that works for both android and java
layout: post
permalink: /2016/02/20/how-to-use-optional-on-android-and-java/
image: assets/images/optional_android_java_featured.png
comments: false
featured: false
hidden: false
categories: [ android, mobile, java, architecture, reactive, fp, oop, functional, programming, engineering ]
tags:
  - android
  - androiddev
  - development
  - java
  - observable
  - option
  - optional
  - patterns
  - programming
  - reactive programming
  - rxjava
  - software patterns
---
First of all, this is <a href="https://dzone.com/articles/guavas-new-optional-class" target="_blank">not a new topic</a> and a lot has already been discussed about it.

With that being said, in this article, I want to explain what ```Optional<T>``` is, expose a few use case scenarios, compare different alternatives (in other languages) and finally, I do want to show you how we can effectively make use of the (<a href="https://github.com/android10/arrow" target="_blank">inexistent for now</a>) ```Optional<T>``` API on Android (although this can be applied to any Java project, specially those ones targeting Java 7).

To get started, let me quote this (retrieved from the <a href="http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html" target="_blank">Official Java 8 documentation</a>): 

>A wise man once said you are not a real Java programmer until you have dealt with a null pointer exception, which is the source of many problems because it is often used to denote the absence of a value.

Although this statement is true and Java 8 documentation refers to the use of **Optional&lt;T&gt;** as **NullPointerException saver**, in my opinion, it is not only useful to minimize the impact of NPE, but to **create more meaningful and readable APIs.**

Additionally, it is well known that not being careful when using null values can lead to a variety of bugs, and for instance, **null is ambiguos, and we do not always have a clear meaning for it:** is it an inexistent value? For example, when a ```Map.get()``` method returns null, can mean the value is absent or the value is present and null.

We will try to answer to these questions in this little journey. Let's get our hands dirty!


## What is an Optional?

First definition from <a href="http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html" target="_blank">Java 8 documentation</a>: 

>Optional object is used to represent null with absent value. It provides various utility methods to facilitate code to handle values as ‘available’ or ‘not available’ instead of checking null values.

This is another similar definition from the <a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Optional.html" target="_blank">Official Guava documentation</a>: 

>An immutable object that may contain a non-null reference to another object. Each instance of this type either contains a non-null reference, or contains nothing (in which case we say that the reference is "absent"); it is never said to "contain null". A non-null Optional&lt;T&gt; reference can be used as a replacement for a nullable T reference. It allows you to represent "a T that must be present" and a "a T that might be absent" as two distinct types in your program, which can aid clarity.

In a nutshell, **the Optional Type API provides a container object which is used to contain not-null objects.**

Let's see a quick example so you get a better understanding of what I am talking about:

```java
Optional<Integer> possible = Optional.of(5);
possible.isPresent();   // returns true
possible.get();         // returns 5
```

**As you can see, we are wrapping an object of type &lt;T&gt; inside an Optional&lt;T&gt; so we can check its existence later.** In other words, **Optional&lt;T&gt;** forces you to care about the value, since in order to retrieve it, you have to call the ```get()``` method (as a good practice always check the presence of it first or return a default value). Just to be clear, we are using <a href="https://github.com/google/guava" target="_blank">Guava</a>'s **Optional&lt;T&gt;** here. 

Don't worry much if you still don't understand, we will explore more afterwards.


## Java 8, Scala, Groovy and Kotlin Optional/Option APIs

As I mentioned above, in this article we will focus on <a href="https://github.com/google/guava" target="_blank">Guava</a> **Optional&lt;T&gt;**, although it is worth to give a quick view to what other programming languages have to offer.

Let's have a look at what <a href="http://docs.groovy-lang.org/latest/html/documentation/index.html#groovy-operators" target="_blank">Groovy</a> and <a href="https://kotlinlang.org/docs/reference/null-safety.html" target="_blank">Kotlin</a> bring up. 

These 2 languages offer similar approaches for **null safety:** '<a href="https://en.wikipedia.org/wiki/Elvis_operator" target="_blank">Elvis Operator</a>'. 

They have added some syntactic sugar and syntax look similar in both of them. 

Let's check this piece of Kotlin code: when we have a nullable reference r, we can say “**if r is not null, use it, otherwise use some non-null value x**”:

```kotlin
val l: Int = if (b != null) b.length else -1
```

Along with the complete if-expression, this can be expressed with the <a href="https://en.wikipedia.org/wiki/Elvis_operator" target="_blank">Elvis Operator</a>, written ?::

```groovy
val l = b?.length ?: -1
```

**If the expression to the left of ?: is not null, the elvis operator returns it, otherwise it returns the expression to the right.** Note that the right-hand side expression is evaluated only if the left-hand side is null. 

For the record, Kotlin also has a check for null conditions at compilation time.

You can dive deeper by checking the official documentation and, by the way I'm not neither a Groovy nor Kotlin guy, so I will leave this to the experts :).


On both <a href="http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html" target="_blank">Java 8</a> and <a href="http://alvinalexander.com/scala/using-scala-option-some-none-idiom-function-java-null" target="_blank">Scala</a> sides we find a <a href="https://mttkay.github.io/blog/2014/01/25/your-app-as-a-function/" target="_blank">monadic</a> approach for Optional&lt;T&gt; (Java) and Option[T] (Scala), allowing us to use <a href="http://fernandocejas.com/2015/01/11/rxjava-observable-tranformation-concatmap-vs-flatmap/" target="_blank">flatMap()</a>, <a href="http://fernandocejas.com/2015/01/11/rxjava-observable-tranformation-concatmap-vs-flatmap/" target="_blank">map()</a>, etc. This means we can compose data stream using Optional&lt;T&gt; in a Functional Programming style. 

Kotlin also offers an **OptionIF&lt;T&gt;** monad with the same purpose.

Let's have a quick look at this Scala example from <a href="http://seanparsons.github.io/scalawat/" target="_blank">Sean Parsons</a> for a better understanding:

```scala
case class Player(name: String)

def lookupPlayer(id: Int): Option[Player] = {
  if (id == 1) Some(new Player("Sean"))
  else if(id == 2) Some(new Player("Greg"))
  else None
}

def lookupScore(player: Player): Option[Int] = {
  if (player.name == "Sean") Some(1000000) else None
}

println(lookupPlayer(1).map(lookupScore))  // Some(Some(1000000))
println(lookupPlayer(2).map(lookupScore))  // Some(None)
println(lookupPlayer(3).map(lookupScore))  // None

println(lookupPlayer(1).flatMap(lookupScore))  // Some(1000000)
println(lookupPlayer(2).flatMap(lookupScore))  // None
println(lookupPlayer(3).flatMap(lookupScore))  // None
```

Last but not least we have **Optional&lt;T&gt;** from <a href="https://github.com/google/guava/" target="_blank">Guava</a>. In favor of it, let's say that its simplified API fits perfectly the Java 7 model: there is only one way to use it which is the imperative one, since in fact, was developed for Java that lacks first-class functions.

I guess so far so good, but there is no Android Java 7 sample code... Ok, you are right, but you will have to keep on reading, so be patient, there is more coming up. 

Also if you are wondering whether in order to use it on Android you will have to compile Guava and its 20k methods, the answer is NO, <a href="https://github.com/android10/arrow" target="_blank">there is an alternative</a> **to bring up Optional&lt;T&gt; to the game.**


## How can we use Optional&lt;T&gt; in Android?

First point to raise here is that we are stuck to Java 7 so **there is no built-in Optional&lt;T&gt;** and we have to aks for help to <a href="https://github.com/android10/arrow" target="_blank">3rd party libraries</a> unfortunately...

Our first player is <a href="https://github.com/google/guava/" target="_blank">Guava</a>, which for Android might not be a good catch, specially (as mentioned above) because of the 20k methods that brings up to your .apk (I'm pretty sure you have heard of <a href="https://developers.soundcloud.com/blog/congratulations-you-have-a-lot-of-code-remedying-androids-method-limit-part-1" target="_blank">65k method limit issue</a> ;)).

The second option is to use <a style="color: #333399;" href="https://github.com/android10/arrow" target="_blank">Arrow</a>, **which is a lightweight open source library I created by gathering and including useful stuff I use in my day to day Android development plus some other utilities I wrote myself, such as annotations for code decoration, etc.** <a href="https://github.com/android10/arrow" target="_blank">You can check the project, documentation and features on Github</a>. 

One thing to remark and shout loudly is that **ALL CREDITS GO TO THE CREATORS OF THESE AWESOME APIs.**


## How do we create Optional&lt;T&gt;?

The **Optional&lt;T&gt; API** is pretty straightforward:

![optional_android_java](/assets/images/optional_android_java_01.png)

Here are the **Optional&lt;T&gt;** query methods:

![optional_android_java](/assets/images/optional_android_java_02.png)

It is time for code samples and use cases so don't leave the room yet.


## Case scenario #1

This is a well known historical <a href="https://en.wikipedia.org/wiki/Tony_Hoare" target="_blank">Tony Hoare</a>'s phrase when he created the null reference:

>I call it my billion-dollar mistake. It was the invention of the null reference in 1965. I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement.

```java
public class Car {
  private final String brand;
  private final String model;
  private final String registrationNumber;

  public Car(String brand, String model, String registrationNumber) {
    this.brand = brand;
    this.model = model;
    this.registrationNumber = registrationNumber;
  }

  public String information() {
    final StringBuilder builder = new StringBuilder();
    builder.append("Model: ").append(model);
    builder.append("Brand: ").append(brand);
    if (registrationNumber != null) {
      builder.append("Registration Number: ").append(registrationNumber);
    }
    return builder.toString();
  }
}
```

The main issue with the following code is that relies on a null reference to indicate the absence of a registration number (a bad practice), so **we can fix this by using Optional&lt;T&gt;** and we print according whether or not the value is present:

```java
public class Car {
  ...
  private final Optional<String> registrationNumber;

  public Car(String brand, String model, String registrationNumber) {
    ...
    this.registrationNumber = Optional.fromNullable(registrationNumber);
  }

  public String information() {
  ...
    if (registrationNumber.isPresent()) {
      builder.append("Registration Number: ").append(registrationNumber.get());
    }
    return builder.toString();
  }
}
```

**The most obvious use case is for avoiding meaningless nulls.** <a href="https://github.com/android10/java-code-examples/blob/master/src/main/java/com/fernandocejas/java/samples/optional/UseCaseScenario02.java" target="_blank">Check full class implementation on Github</a>. Let's move forward to the next scenario.


## Case scenario #2

Let's say we need to parse a JSON file coming from an API response (something very common in Mobile Development). 

In this case we can use **Optional&lt;T&gt;** in our Entity ir order to force the client to care about the existence of the value before using or doing anything with it.

Check out "nickname" field and getter in the following sample code:

```java
public class User {
  @SerializedName("id")
  private int userId;

  @SerializedName("full_name")
  private String fullname;

  @SerializedName("nickname")
  private String nickname;

  public int id() {
    return userId;
  }

  public String fullname() {
    return fullname;
  }

  public Optional<String> nickname() {
    return Optional.fromNullable(nickname);
  }
}
```

<a href="https://github.com/android10/java-code-examples/blob/master/src/main/java/com/fernandocejas/java/samples/optional/UseCaseScenario02.java" target="_blank">Complete sample class on Github</a>.


## Case scenario #3

This is another use case we usually stumble upon at <a href="https://developers.soundcloud.com/blog" target="_blank">@SoundCloud</a> in our Android application.

When we need to construct our feed or any list of items and show them at UI level (presentation models), we have items coming from different data sources, and some of them might be **Optional&lt;T&gt;**, like for example, a Facebook invitation, a promoted track, etc.

Check this little example, which tries to emulate the above situation in a very simplified way (for learning purpose) using RxJava:

```java
public class Sample {

  public static final Func1<Optional<List<String>>, Observable<List<String>>> TO_AD_ITEM =
      ads -> ads.isPresent()
          ? Observable.just(ads.get())
          : Observable.just(Collections.<String>emptyList());

  public static final Func1<List<String>, Boolean> EMPTY_ELEMENTS = ads -> !ads.isEmpty();

  public Observable<List<String>> feed() {
    return ads()
        .flatMap(TO_AD_ITEM)
        .filter(EMPTY_ELEMENTS)
        .concatWith(tracks())
        .observeOn(Schedulers.immediate());
  }

  private Observable<Optional<List<String>>> ads() {
    return Observable.just(Optional.fromNullable(
      Collections.singletonList("This is and Ad")));
  }

  private Observable<List<String>> tracks() {
    return Observable.just(
      Arrays.asList("IronMan Song", "Wolverine Song", "Batman Sound"));
  }
}
```

The most important part here is that when we combine both **Observables&lt;T&gt;** (```tracks()``` and ```ads()``` method) we use <a href="http://fernandocejas.com/2015/01/11/rxjava-observable-tranformation-concatmap-vs-flatmap/" target="_blank">flatMap()</a> and <a href="http://reactivex.io/documentation/operators/filter.html" target="_blank">filter()</a> operators to determine whether or not we are gonna emit ads and for instance, display them at UI level (I'm using Java 8 lambdas here to make the code more readable):

```java
public static final Func1<Optional<List<String>>, Observable<List<String>>> TO_AD_ITEM =
      ads -> ads.isPresent()
          ? Observable.just(ads.get())
          : Observable.just(Collections.<String>emptyList());

public static final Func1<List<String>, Boolean> EMPTY_ELEMENTS = ads -> !ads.isEmpty();
```

<a href="https://github.com/android10/java-code-examples/blob/master/src/main/java/com/fernandocejas/java/samples/optional/UseCaseScenario03.java" target="_blank">Check out the full implementation on Github</a>.


## Conclusion

To wrap up, **in software development there are no silver bullets and as programmers we tend to overthink and overuse things so don't pollute your code with Optional&lt;T&gt; everywhere, use them carefully where it makes sense.**

Also let me quote Joshua Bloch in his talk '<a href="http://www.infoq.com/articles/API-Design-Joshua-Bloch" target="_blank">How to Design a Good API and Why it Matters</a>':

>APIs should be easy to use and hard to misuse: It should be easy to do simple things; possible to do complex things; and impossible, or at least difficult, to do wrong things.

I completely agree with this, and from an API design standpoint, **Optional&lt;T&gt;** is a good example of a well design API: it will help you address and protect from **NullPointerException** issues (although not fully eliminate them), write concise and readable code and additionally will provide a more meaningful codebase.


## Sample Code

You can find all the sample code in a Github repo I created for this purpose:

* <a href="https://github.com/android10/java-code-examples" target="_blank">https://github.com/android10/java-code-examples</a> 

Also visit the **Arrow** project repo to make use of **Optional&lt;T&gt;** in Android:

* <a href="https://github.com/android10/arrow" target="_blank">https://github.com/android10/arrow</a>


## References:

* <a href="http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html" target="_blank">http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html</a>
* <a href="https://github.com/google/guava/wiki/UsingAndAvoidingNullExplained" target="_blank">https://github.com/google/guava/wiki/UsingAndAvoidingNullExplained</a>
* <a href="https://dzone.com/articles/guavas-new-optional-class" target="_blank">https://dzone.com/articles/guavas-new-optional-class</a>
* <a href="https://kerflyn.wordpress.com/2011/12/05/from-optional-to-monad-with-guava/" target="_blank">https://kerflyn.wordpress.com/2011/12/05/from-optional-to-monad-with-guava/</a>
* <a href="http://techblog.bozho.net/the-optional-type/" target="_blank">http://techblog.bozho.net/the-optional-type/</a>
* <a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Optional.html" target="_blank">http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Optional.html</a>
* <a href="http://seanparsons.github.io/scalawat/Using+flatMap+With+Option.html" target="_blank">http://seanparsons.github.io/scalawat/Using+flatMap+With+Option.html</a>
* <a href="http://www.nurkiewicz.com/2013/05/null-safety-in-kotlin.html" target="_blank">http://www.nurkiewicz.com/2013/05/null-safety-in-kotlin.html</a>
* <a href="https://kotlinlang.org/docs/reference/null-safety.html" target="_blank">https://kotlinlang.org/docs/reference/null-safety.html</a>