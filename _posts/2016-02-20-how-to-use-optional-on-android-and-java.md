---
id: 9
title: How to use Optional values on Java and Android
date: 2016-02-20T23:19:45+00:00
author: Fernando Cejas
layout: post
permalink: /2016/02/20/how-to-use-optional-on-android-and-java/
categories:
  - Android
  - Development
  - Java
  - Reactive Programming
  - Software Architecture
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
<p style="text-align: justify;">
  First of all, this is <a href="https://dzone.com/articles/guavas-new-optional-class" target="_blank">not a new topic</a> and a lot has already been discussed about it.<br /> <span style="text-decoration: underline;">With that being said</span>, in this article, I want to explain what <strong><span style="color: #333399;">Optional<T></span></strong> is, expose a few use case scenarios, compare different alternatives (in other languages) and finally, I do want to show you how we can <span style="text-decoration: underline;">effectively</span> make use of the (<a href="https://github.com/android10/arrow" target="_blank">inexistent for now</a>) <strong><span style="color: #333399;">Optional<T></span> <span style="color: #333399;">API</span></strong> on Android (although this can be applied to any <strong><span style="color: #333399;">Java</span></strong> project, specially those ones targeting <span style="color: #333399;"><strong>Java 7</strong></span>).
</p>

<p style="text-align: justify;">
  To get started, let me quote this (retrieved from the <a href="http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html" target="_blank">Official Java 8 documentation</a>):<br /> <strong><span style="color: #333399;">&#8220;A wise man once said you are not a real Java programmer until you&#8217;ve dealt with a null pointer exception, which is the source of many problems because it is often used to denote the absence of a value&#8221;.</span></strong>
</p>

<p style="text-align: justify;">
  Although this statement is true and Java 8 documentation refers to the use of <strong><span style="color: #333399;">Optional<T></span></strong> as <span style="color: #333399;"><strong>NullPointerException saver</strong></span>, in my opinion, it is not only useful to minimize the impact of NPE, but to create more <strong><span style="color: #333399;">meaningful and readable APIs</span></strong>.
</p>

<p style="text-align: justify;">
  Additionally, it is well known that not being careful when using null values can lead to a variety of bugs, and for instance, <span style="color: #333399;"><strong>null is ambiguos</strong></span>, and we don&#8217;t always have a clear meaning for it: is it an inexistent value? For example, when a <em>Map.get()</em> method returns <em>null</em>, can mean the value is <em>absent</em> or the value is <em>present</em> and <em>null</em>.
</p>

<p style="text-align: justify;">
  We will try to answer to these questions in this little journey. Let&#8217;s get our hands dirty then!
</p>

### What is an Optional?

<p style="text-align: justify;">
  First definition from <a href="http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html" target="_blank">Java 8 documentation</a>:<br /> <span style="color: #333399;"><strong>&#8220;Optional object is used to represent null with absent value. It provides various utility methods to facilitate code to handle values as ‘available’ or ‘not available’ instead of checking null values&#8221;.</strong></span>
</p>

<p style="text-align: justify;">
  This is another similar definition from the <a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Optional.html" target="_blank">Official Guava documentation</a>:<br /> <span style="color: #333399;"><strong>&#8220;An immutable object that may contain a non-null reference to another object. Each instance of this type either contains a non-null reference, or contains nothing (in which case we say that the reference is &#8220;absent&#8221;); it is never said to &#8220;contain null&#8221;. A non-null Optional<T> reference can be used as a replacement for a nullable T reference. It allows you to represent &#8220;a T that must be present&#8221; and a &#8220;a T that might be absent&#8221; as two distinct types in your program, which can aid clarity&#8221;.</strong></span>
</p>

<p style="text-align: justify;">
  <span style="text-decoration: underline;">In a nutshell, the Optional Type API provides a container object which is used to contain not-null objects. </span>Let&#8217;s see a quick example so you get a better understanding of what I am talking about:
</p>

<pre class="font-size:13 lang:java decode:true " title="Simple Optional Sample">Optional&lt;Integer&gt; possible = Optional.of(5);
possible.isPresent(); 	// returns true
possible.get(); 	// returns 5</pre>

<p style="text-align: justify;">
  As you can see we are wrapping an object of type <strong><span style="color: #333399;"><T></span></strong> inside an <strong><span style="color: #333399;">Optional<T></span></strong> so we can check its existence later. <strong><span style="color: #333399;">In other words, Optional<T> forces you to care about the value, since in order to retrieve it, you have to call the get() method</span></strong> (<span style="text-decoration: underline;">as a good practice always check the presence of it first or return a default value</span>). Just to be clear, we are using <a href="https://github.com/google/guava" target="_blank">Guava</a>&#8216;s <strong><span style="color: #333399;">Optional<T></span></strong> here.<br /> Don&#8217;t worry much if you still don&#8217;t understand, we will explore more afterwards.
</p>

### Java 8, Scala, Groovy and Kotlin Optional/Option APIs

<p style="text-align: justify;">
  As I mentioned above, in this article we will focus on <a href="https://github.com/google/guava" target="_blank">Guava</a> <span style="color: #333399;"><strong>Optional<T></strong></span>, although it is worth to give a quick view to what other programming languages have to offer.
</p>

<p style="text-align: justify;">
  Let&#8217;s have a look at what <a href="http://docs.groovy-lang.org/latest/html/documentation/index.html#groovy-operators" target="_blank">Groovy</a> and <a href="https://kotlinlang.org/docs/reference/null-safety.html" target="_blank">Kotlin</a> bring up. These 2 languages offer similar approaches for null safety: &#8216;<a href="https://en.wikipedia.org/wiki/Elvis_operator" target="_blank">Elvis Operator</a>&#8216;. They have added some syntactic sugar and syntax look similar in both of them. <span style="color: #333399;"><strong>Let&#8217;s check this piece of Kotlin code</strong></span>: <span style="color: #333399;"><strong>when we have a nullable reference r, we can say “if r is not null, use it, otherwise use some non-null value x”:</strong></span>
</p>

<pre class="font-size:13 lang:java decode:true" title="Elvis Operator">val l: Int = if (b != null) b.length else -1</pre>

<p style="text-align: justify;">
  Along with the complete if-expression, this can be expressed with the <a href="https://en.wikipedia.org/wiki/Elvis_operator" target="_blank">Elvis Operator</a>, written <strong><span style="color: #333399;">?:</span></strong>:
</p>

<pre class="font-size:13 lang:java decode:true" title="Elvis Operator">val l = b?.length ?: -1</pre>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>If the expression to the left of ?: is not null, the elvis operator returns it, otherwise it returns the expression to the right.</strong></span> Note that the right-hand side expression is evaluated only if the left-hand side is null. For the record, Kotlin also has a check for null conditions at compilation time.<br /> You can dive deeper by checking the official documentation and, by the way <span style="text-decoration: underline;">I&#8217;m not neither a Groovy nor Kotlin guy</span>, so I will leave this to the experts :).
</p>

<p style="text-align: justify;">
  On both <a href="http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html" target="_blank">Java 8</a> and <a href="http://alvinalexander.com/scala/using-scala-option-some-none-idiom-function-java-null" target="_blank">Scala</a> sides we find a <a href="https://mttkay.github.io/blog/2014/01/25/your-app-as-a-function/" target="_blank">monadic</a> approach for <strong><span style="color: #333399;">Optional<T></span></strong> (Java) and <span style="color: #333399;"><strong>Option[T]</strong></span> (Scala), allowing us to use <a href="http://fernandocejas.com/2015/01/11/rxjava-observable-tranformation-concatmap-vs-flatmap/" target="_blank">flatMap()</a>, <a href="http://fernandocejas.com/2015/01/11/rxjava-observable-tranformation-concatmap-vs-flatmap/" target="_blank">map()</a>, etc. <strong><span style="color: #333399;">This means we can compose data stream using Optional<T> in a Functional Programming style. Kotlin also offers an OptionIF<T> monad with the same purpose.</span></strong><br /> Let&#8217;s have a quick look at this Scala example from <a href="http://seanparsons.github.io/scalawat/" target="_blank">Sean Parsons</a> for a better understanding:
</p>

<pre class="font-size:13 lang:scala decode:true" title="Option value in Scala">case class Player(name: String)
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
println(lookupPlayer(3).flatMap(lookupScore))  // None</pre>

<p style="text-align: justify;">
  Last but not least we have <strong><span style="color: #333399;">Optional<T></span></strong> from <a href="https://github.com/google/guava/" target="_blank">Guava</a>. In favor of it, let&#8217;s say that its simplified API fits perfectly the Java 7 model: <strong><span style="color: #333399;">there is only one way to use it which is the imperative one, since in fact, was developed for Java that lacks first-class functions</span></strong>.
</p>

<p style="text-align: justify;">
  I guess so far so good, but there is no Android Java 7 sample code&#8230; Ok, you are right, but <strong><span style="color: #333399;">you will have to keep on reading</span></strong>, so be patient, there is more coming up. Also if you are wondering whether in order to use it on Android you will have to compile Guava and its 20k methods, the answer is NO, <a href="https://github.com/android10/arrow" target="_blank">there is an alternative</a> to bring up <strong><span style="color: #333399;">Optional<T></span></strong> to the game.
</p>

### How can we use Optional<T> in Android?

<p style="text-align: justify;">
  First point to raise here is that <strong><span style="color: #333399;">we are stuck to Java 7</span></strong> so there is no built-in <strong><span style="color: #333399;">Optional<T></span></strong> and we have to aks for help to <a href="https://github.com/android10/arrow" target="_blank">3rd party libraries</a> unfortunately&#8230;<br /> Our first player is <a href="https://github.com/google/guava/" target="_blank">Guava</a>, <span style="color: #333399;"><strong>which for Android might not be a good catch</strong></span>, specially (as mentioned above) because of the 20k methods that brings up to your <strong><span style="color: #333399;">.apk</span></strong> (I&#8217;m pretty sure you have heard of <a href="https://developers.soundcloud.com/blog/congratulations-you-have-a-lot-of-code-remedying-androids-method-limit-part-1" target="_blank">65k method limit issue</a> ;)).
</p>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>The second option is to use <a style="color: #333399;" href="https://github.com/android10/arrow" target="_blank">Arrow</a>,</strong></span> which is an lightweight open source library I created by gathering and including useful stuff I use in my day to day Android development plus some other utilities I wrote myself, such as annotations for code decoration, etc. <a href="https://github.com/android10/arrow" target="_blank">You can check the project, documentation and features on Github</a>. One thing to remark and shout loudly is that <strong><span style="color: #333399;">ALL CREDITS GO TO THE CREATORS OF THESE AWESOME APIs.</span></strong>
</p>

### How do we create Optional<T>?

<p style="text-align: justify;">
  The <strong><span style="color: #333399;">Optional<T></span></strong> API is pretty straightforward:
</p>

<img class="aligncenter wp-image-454 size-full" title="Optional Creation" src="http://fernandocejas.com/wp-content/uploads/2016/02/optional_creation-2.png" alt="" width="657" height="382" srcset="http://fernandocejas.com/wp-content/uploads/2016/02/optional_creation-2.png 657w, http://fernandocejas.com/wp-content/uploads/2016/02/optional_creation-2-300x174.png 300w" sizes="(max-width: 657px) 100vw, 657px" />

<p style="text-align: justify;">
  Here are the <strong><span style="color: #333399;">Optional<T></span></strong> query methods:
</p>

<img class="aligncenter wp-image-449 size-full" src="http://fernandocejas.com/wp-content/uploads/2016/02/optional_query.png" alt="optional_query" width="652" height="592" srcset="http://fernandocejas.com/wp-content/uploads/2016/02/optional_query.png 652w, http://fernandocejas.com/wp-content/uploads/2016/02/optional_query-300x272.png 300w" sizes="(max-width: 652px) 100vw, 652px" />

<p style="text-align: justify;">
  It is time for code samples and use cases so don&#8217;t leave the room yet.
</p>

### Case scenario #1

<p style="text-align: justify;">
  This is a well known historical <a href="https://en.wikipedia.org/wiki/Tony_Hoare" target="_blank">Tony Hoare</a>&#8216;s phrase when he created the <em>null</em> reference:<br /> <strong><span style="color: #333399;">&#8220;I call it my billion-dollar mistake. It was the invention of the null reference in 1965. I couldn&#8217;t resist the temptation to put in a null reference, simply because it was so easy to implement.&#8221;</span></strong>
</p>

<pre class="font-size:13 lang:java decode:true" title="Car sample">public class Car {
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
}</pre>

<p style="text-align: justify;">
  The main issue with the following code is that relies on a <em>null</em> reference to indicate the absence of a registration number (<span style="text-decoration: underline;">a bad practice</span>), so <strong><span style="color: #333399;">we can fix this by using Optional<T> and we print according whether or not the value is present:</span></strong>
</p>

<pre class="font-size:13 lang:java decode:true" title="Car sample">public class Car {
  ...
  private final Optional&lt;String&gt; registrationNumber;

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
}</pre>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>The most obvious use case is for avoiding meaningless nulls</strong></span>. <a href="https://github.com/android10/java-code-examples/blob/master/src/main/java/com/fernandocejas/java/samples/optional/UseCaseScenario02.java" target="_blank">Check full class implementation on Github</a>.<br /> Let&#8217;s move forward to the next scenario.
</p>

### Case scenario #2

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>Let&#8217;s say we need to parse a JSON file coming from an API response</strong></span> (<span style="text-decoration: underline;">something very common in Mobile Development</span>). In this case we can use <span style="color: #333399;"><strong>Optional<T></strong></span> in our Entity ir order to <strong><span style="color: #333399;">force the client to care about the existence of the value before using or doing anything with it.</span></strong><br /> Check out <em>&#8220;nickname&#8221;</em> field and getter in the following sample code:
</p>

<pre class="font-size:13 lang:java decode:true " title="JSON parsing sample">public class User {
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

  public Optional&lt;String&gt; nickname() {
    return Optional.fromNullable(nickname);
  }
}</pre>

<a href="https://github.com/android10/java-code-examples/blob/master/src/main/java/com/fernandocejas/java/samples/optional/UseCaseScenario02.java" target="_blank">Complete sample class on Github</a>.

### Case scenario #3

<p style="text-align: justify;">
  This is another use case we usually stumble upon at <a href="https://developers.soundcloud.com/blog" target="_blank">@SoundCloud</a> in our Android application.<br /> <span style="color: #333399;"><strong>When we need to construct our feed or any list of items and show them at UI level (presentation models), we have items coming from different data sources, and some of them might be Optional<T>, like for example, a Facebook invitation, a promoted track, etc.</strong></span>
</p>

<p style="text-align: justify;">
  Check this little example, which tries to emulate the above situation in a very simplified way (for learning purpose) using <span style="color: #333399;"><strong>RxJava</strong></span>:
</p>

<pre class="font-size:13 lang:java decode:true" title="RxJava sample">public class Sample {

  public static final Func1&lt;Optional&lt;List&lt;String&gt;&gt;, Observable&lt;List&lt;String&gt;&gt;&gt; TO_AD_ITEM =
      ads -&gt; ads.isPresent()
          ? Observable.just(ads.get())
          : Observable.just(Collections.&lt;String&gt;emptyList());

  public static final Func1&lt;List&lt;String&gt;, Boolean&gt; EMPTY_ELEMENTS = ads -&gt; !ads.isEmpty();

  public Observable&lt;List&lt;String&gt;&gt; feed() {
    return ads()
        .flatMap(TO_AD_ITEM)
        .filter(EMPTY_ELEMENTS)
        .concatWith(tracks())
        .observeOn(Schedulers.immediate());
  }

  private Observable&lt;Optional&lt;List&lt;String&gt;&gt;&gt; ads() {
    return Observable.just(Optional.fromNullable(Collections.singletonList("This is and Ad")));
  }

  private Observable&lt;List&lt;String&gt;&gt; tracks() {
    return Observable.just(Arrays.asList("IronMan Song", "Wolverine Song", "Batman Sound"));
  }
}</pre>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>The most important part here is that when we combine both Observables<T></strong></span> (<em>tracks()</em> and <em>ads()</em> method) we use <a href="http://fernandocejas.com/2015/01/11/rxjava-observable-tranformation-concatmap-vs-flatmap/" target="_blank">flatMap()</a> and <a href="http://reactivex.io/documentation/operators/filter.html" target="_blank">filter()</a> operators to determine whether or not we are gonna emit ads and for instance, display them at UI level (<span style="text-decoration: underline;">I&#8217;m using Java 8 lambdas here to make the code more readable</span>):
</p>

<pre class="font-size:13 lang:java decode:true " title="RxJava sample">public static final Func1&lt;Optional&lt;List&lt;String&gt;&gt;, Observable&lt;List&lt;String&gt;&gt;&gt; TO_AD_ITEM =
      ads -&gt; ads.isPresent()
          ? Observable.just(ads.get())
          : Observable.just(Collections.&lt;String&gt;emptyList());

public static final Func1&lt;List&lt;String&gt;, Boolean&gt; EMPTY_ELEMENTS = ads -&gt; !ads.isEmpty();</pre>

<a href="https://github.com/android10/java-code-examples/blob/master/src/main/java/com/fernandocejas/java/samples/optional/UseCaseScenario03.java" target="_blank">Check out the full implementation on Github</a>.

### Conclusion

<p style="text-align: justify;">
  To wrap up, in software development there are no silver bullets and as programmers we tend to overthink and overuse things so <strong><span style="color: #333399;">don&#8217;t pollute your code with Optional<T> everywhere, use them carefully where it makes sense.</span></strong>
</p>

<p style="text-align: justify;">
  Also let me quote Joshua Bloch in his talk &#8216;<a href="http://www.infoq.com/articles/API-Design-Joshua-Bloch" target="_blank">How to Design a Good API and Why it Matters</a>&#8216;:<br /> <span style="color: #333399;"><strong>&#8220;APIs should be easy to use and hard to misuse: It should be easy to do simple things; possible to do complex things; and impossible, or at least difficult, to do wrong things.&#8221;</strong></span><br /> I completely agree with this, and from an API design standpoint, <strong><span style="color: #333399;">Optional<T></span></strong> is a good example of a well design API: <strong><span style="color: #333399;">it will help you address and protect from NullPointerException issues (although not fully eliminate them), write concise and readable code and additionally will provide a more meaningful codebase.</span></strong>
</p>

### Sample Code

You can find all the <span style="color: #333399;"><strong>sample code in a Github repo</strong></span> I created for this purpose: <a href="https://github.com/android10/java-code-examples" target="_blank">https://github.com/android10/java-code-examples</a> and visit the **<span style="color: #333399;">Arrow project repo</span>** to make use of <span style="color: #333399;"><strong>Optional<T></strong></span> in Android: <a href="https://github.com/android10/arrow" target="_blank">https://github.com/android10/arrow</a>

### References:

<a href="http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html" target="_blank">http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html</a>
  
<a href="https://github.com/google/guava/wiki/UsingAndAvoidingNullExplained" target="_blank">https://github.com/google/guava/wiki/UsingAndAvoidingNullExplained</a>
  
<a href="https://dzone.com/articles/guavas-new-optional-class" target="_blank">https://dzone.com/articles/guavas-new-optional-class</a>
  
<a href="https://kerflyn.wordpress.com/2011/12/05/from-optional-to-monad-with-guava/" target="_blank">https://kerflyn.wordpress.com/2011/12/05/from-optional-to-monad-with-guava/</a>
  
<a href="http://techblog.bozho.net/the-optional-type/" target="_blank">http://techblog.bozho.net/the-optional-type/</a>
  
<a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Optional.html" target="_blank">http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Optional.html</a>
  
<a href="http://seanparsons.github.io/scalawat/Using+flatMap+With+Option.html" target="_blank">http://seanparsons.github.io/scalawat/Using+flatMap+With+Option.html</a>
  
<a href="http://www.nurkiewicz.com/2013/05/null-safety-in-kotlin.html" target="_blank">http://www.nurkiewicz.com/2013/05/null-safety-in-kotlin.html</a>
  
<a href="https://kotlinlang.org/docs/reference/null-safety.html" target="_blank">https://kotlinlang.org/docs/reference/null-safety.html</a>