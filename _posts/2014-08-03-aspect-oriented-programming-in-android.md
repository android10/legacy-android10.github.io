---
id: 3
title: Aspect Oriented Programming in Android
date: 2014-08-03T01:02:51+00:00
author: Fernando Cejas
description: AOP is a paradigm that has been with us for many years, and I found it very useful to apply it to Android. After some investigation I consider that we can get a lot of advantages and very useful stuff when making use of it
layout: post
permalink: /2014/08/03/aspect-oriented-programming-in-android/
categories:
  - Android
  - Aspect Oriented Programming
  - Development
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
  - weaving
---
<p class="justify"><span class="boldtext">Aspect-oriented programming</span> entails breaking down program logic into <span class="boldtext">"concerns"</span> (cohesive areas of functionality). This means, that with <span class="boldtext">AOP</span>, we can add executable blocks to some source code without explicitly changing it. This programming paradigm pretends that <span class="boldtext">“cross-cutting concerns”</span> (the logic needed at many places, without a single class where to implement them) should be implemented once and injected it many times into those places.</p>

<p class="justify"><span class="boldtext">Code injection becomes a very important part of AOP:</span> it is useful for dealing with the mentioned <span class="boldtext">"concerns"</span> that cut across the whole application, such as logging or performance monitoring, and, using it in this way, should not be something used rarely as you might think, quite the contrary; every programmer will come into a situation where this ability of injecting code, could prevent a lot of pain and frustration.</p>

<p class="justify"><span class="boldtext">AOP</span> is a paradigm that has been with us for many years, and I found it very useful to apply it to Android. After some investigation I consider that we can get a lot of advantages and very useful stuff when making use of it.</p>

## Terminology (Mini glossary)

<p class="justify">Before we get started, let's have a look at some vocabulary that we should keep in mind:</p>

  * <p class="justify"><span class="boldtext">Cross-cutting concerns:</span> Even though most classes in an OO model will perform a single, specific function, they often share common, secondary requirements with other classes. For example, we may want to add logging to classes within the data-access layer and also to classes in the UI layer whenever a thread enters or exits a method. Even though each class has a very different primary functionality, the code needed to perform the secondary functionality is often identical.</p>
  * <p class="justify"><span class="boldtext">Advice:</span> The code that is injected to a class file. Typically we talk about before, after, and around advices, which are executed before, after, or instead of a target method. It’s possible to make also other changes than injecting code into methods, e.g. adding fields or interfaces to a class.
  * <p class="justify"><span class="boldtext">Joint point:</span> A particular point in a program that might be the target of code injection, e.g. a method call or method entry.</p>
  * <p class="justify"><span class="boldtext">Pointcut:</span> An expression which tells a code injection tool where to inject a particular piece of code, i.e. to which joint points to apply a particular advice. It could select only a single such point – e.g. execution of a single method – or many similar points – e.g. executions of all methods marked with a custom annotation such as @DebugTrace.</p>
  * <p class="justify"><span class="boldtext">Aspect:</span> The combination of the pointcut and the advice is termed an aspect. For instance, we add a logging aspect to our application by defining a pointcut and giving the correct advice.</p>
  * <p class="justify"><span class="boldtext">Weaving: </span>The process of injecting code – advices – into the target places – joint points.</p>

This picture summarizes a bit a few of these concepts:

<img class="aligncenter wp-image-164 size-full" src="/assets/migrated/AspectOrientedProgramming.png" alt="Aspect Oriented Programming" width="674" height="391" srcset="/assets/migrated/AspectOrientedProgramming.png 674w, /assets/migrated/AspectOrientedProgramming-300x174.png 300w" sizes="(max-width: 674px) 100vw, 674px" />

## So...where and when can we apply AOP?

Some examples of cross-cutting concerns are:

  * <span class="boldtext">Logging</span>
  * <span class="boldtext">Persistance</span>
  * <span class="boldtext">Performance monitoring</span>
  * <span class="boldtext">Data Validation</span>
  * <span class="boldtext">Caching</span>
  * <a href="http://en.wikipedia.org/wiki/Cross-cutting_concern" target="_blank">Many others</a>

<p class="justify">And in relation with "when the magic happens", the code can be injected at different points in time:</p>

  * <p class="justify"><span class="boldtext">At run-time:</span> your code has to explicitly ask for the enhanced code, e.g. by using a Dynamic Proxy (this is arguably not true code injection). Anyway here is an example I created for testing it.</p>
  * <p class="justify"><span class="boldtext">At load-time:</span> the modification are performed when the target classes are being loaded by Dalvik or ART. Byte-code or Dex-code weaving.</p>
  * <p class="justify"><span class="boldtext">At build-time:</span> you add an extra step to your build process to modify the compiled classes before packaging and deploying your application. Source-code weaving.</p>

<p class="justify">Depending on the situation you will be choosing one or the other :).</p>

## Tools and Libraries

<span class="boldtext">There are a few tools and libraries out there that help us use AOP:</span>

  * <p class="justify"><span class="boldtext"><a href="https://eclipse.org/aspectj/" target="_blank">AspectJ:</a></span> A seamless aspect-oriented extension to the Javatm programming language (works with Android).</p>
  * <p class="justify"><span class="boldtext"><a href="https://github.com/crimsonwoods/javassist-android" target="_blank">Javassist for Android:</a></span> An android porting of the very well known java library Javassist for bytecode manipulation.</p>
  * <p class="justify"><span class="boldtext"><a href="https://code.google.com/p/dexmaker/" target="_blank">DexMaker:</a></span> A Java-language API for doing compile time or runtime code generation targeting the Dalvik VM.</p>
  * <p class="justify"><span class="boldtext"><a href="http://asm.ow2.org/asmdex-index.html" target="_blank">ASMDEX:</a></span> A bytecode manipulation library as ASM but it handles the DEX bytecode used by Android executables.</p>

## Why AspectJ?

For our example below I have chosen AspectJ for the following reasons:

  * <span class="boldtext">Very powerful.</span>
  * <span class="boldtext">Supports build time and load time code injection.</span>
  * <span class="boldtext">Easy to use.</span>

## Example

<p class="justify">Let's say we want to measure the performance of a method (how long takes its execution). For doing this we want to mark our method with a <span class="boldtext">@DebugTrace</span> annotation and want to see the results using the logcat transparently without having to write code in each annotated method. <span class="boldtext">Our approach is to use AspectJ for this purpose.</span></p>
  
<p class="justify">This is what is gonna happen under the hood:</p>

  * <span class="boldtext">The annotation will be processed in a new step we are adding to our compilation fase.</span>
  * <span class="boldtext">Necessary boilerplate code will be generated and injected in the annotated method.</span>

<p class="justify">I have to say here that while I was researching I found <a href="https://github.com/JakeWharton/hugo" target="_blank">Jake Wharton's Hugo Library</a> that it is suppose to do the same, so I refactored my code and looks similar to it, although mine is a more primitive and simpler version (I have learnt a lot by looking at its code by the way).</p>

<img class="aligncenter size-full wp-image-165" src="/assets/migrated/AspectWeaving.png" alt="AspectWeaving" width="676" height="358" srcset="/assets/migrated/AspectWeaving.png 676w, /assets/migrated/AspectWeaving-300x158.png 300w" sizes="(max-width: 676px) 100vw, 676px" />

## Project structure

<p class="justify">We will break up our sample application into <span class="boldtext">2 modules,</span> the first will contain our android app and the second will be an android library that will make use of AspectJ library for weaving (code injection).</p>
  
<p class="justify">You may be wondering why we are using an android library module instead of a pure java library: <span class="boldtext">the reason is that for AspectJ to work on Android we have to make use of some hooks when compiling our app and this is only possible using the android-library gradle plugin.</span> (Do not worry about this yet, cause I will be giving some more details later).</p>

## Creating our annotation

<p class="justify">We first create our Java annotation. This annotation will be persisted in the class (<span class="boldtext">RetentionPolicy.CLASS</span>) file and we will be able to annotate any constructor or method with it (<span class="boldtext">ElementType.CONSTRUCTOR and ElementType.METHOD</span>). So our <span class="boldtext">DebugTrace.java</span> file will look like this:</p>

```java
@Retention(RetentionPolicy.CLASS)
@Target({ ElementType.CONSTRUCTOR, ElementType.METHOD })
public @interface DebugTrace {}
```

## Our StopWatch for performance monitoring

<p class="justify">I have created a simple class that encapsulates time start/stop. Here is our <span class="boldtext">StopWatch.java</span> class:</p>

```java
/**
 * Class representing a StopWatch for measuring time.
 */
public class StopWatch {
  private long startTime;
  private long endTime;
  private long elapsedTime;

  public StopWatch() {
    //empty
  }

  private void reset() {
    startTime = 0;
    endTime = 0;
    elapsedTime = 0;
  }

  public void start() {
    reset();
    startTime = System.nanoTime();
  }

  public void stop() {
    if (startTime != 0) {
      endTime = System.nanoTime();
      elapsedTime = endTime - startTime;
    } else {
      reset();
    }
  }

  public long getTotalTimeMillis() {
    return (elapsedTime != 0) ? TimeUnit.NANOSECONDS.toMillis(endTime - startTime) : 0;
  }
}
```

## DebugLog Class

<p class="justify">I just decorated the <span class="boldtext">"android.util.Log"</span> cause my first idea was to add some more functionality to the android log. Here it is:</p>

```java
/**
 * Wrapper around {@link android.util.Log}
 */
public class DebugLog {

  private DebugLog() {}

  /**
   * Send a debug log message
   *
   * @param tag Source of a log message.
   * @param message The message you would like logged.
   */
  public static void log(String tag, String message) {
    Log.d(tag, message);
  }
}
```

## Our Aspect

<p class="justify">Now it is time to create our aspect class (<span class="boldtext">TraceAspect.java</span>) that will be in charge of managing the annotation processing and source-code weaving.</p>

```java
/**
 * Aspect representing the cross cutting-concern: Method and Constructor Tracing.
 */
@Aspect
public class TraceAspect {

  private static final String POINTCUT_METHOD =
      "execution(@org.android10.gintonic.annotation.DebugTrace * *(..))";

  private static final String POINTCUT_CONSTRUCTOR =
      "execution(@org.android10.gintonic.annotation.DebugTrace *.new(..))";

  @Pointcut(POINTCUT_METHOD)
  public void methodAnnotatedWithDebugTrace() {}

  @Pointcut(POINTCUT_CONSTRUCTOR)
  public void constructorAnnotatedDebugTrace() {}

  @Around("methodAnnotatedWithDebugTrace() || constructorAnnotatedDebugTrace()")
  public Object weaveJoinPoint(ProceedingJoinPoint joinPoint) throws Throwable {
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    String className = methodSignature.getDeclaringType().getSimpleName();
    String methodName = methodSignature.getName();

    final StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    Object result = joinPoint.proceed();
    stopWatch.stop();

    DebugLog.log(className, buildLogMessage(methodName, stopWatch.getTotalTimeMillis()));

    return result;
  }

  /**
   * Create a log message.
   *
   * @param methodName A string with the method name.
   * @param methodDuration Duration of the method in milliseconds.
   * @return A string representing message.
   */
  private static String buildLogMessage(String methodName, long methodDuration) {
    StringBuilder message = new StringBuilder();
    message.append("Gintonic --> ");
    message.append(methodName);
    message.append(" --> ");
    message.append("[");
    message.append(methodDuration);
    message.append("ms");
    message.append("]");

    return message.toString();
  }
}
```

Some important points to mention here:

  * <p class="justify">We declare 2 public methods with 2 pointcuts that will filter all methods and constructors annotated with <span class="boldtext">"org.android10.gintonic.annotation.DebugTrace"</span>.</p>
  * <p class="justify">We define the <span class="boldtext">"weaveJointPoint(ProceedingJoinPoint joinPoint)"</span> annotated with <span class="boldtext">"@Around"</span> which means that our code injection will happen around the annotated method with <span class="boldtext">"@DebugTrace"</span>.</p>
  * <p class="justify">The line <span class="boldtext">"Object result = joinPoint.proceed();"</span> is where the annotated method execution happens, so before this, is where we start our <span class="boldtext">StopWatch</span> to start measuring time, and after that, we stop it.</p>
  * <p class="justify">Finally we build our message and print it using the <span class="boldtext">Android Log</span>.</p>

## Making AspectJ to work with Android

<p class="justify">Now everything should be working, but, if we compile our sample, we will see that nothing happens.</p>
  
<p class="justify"><span class="boldtext">The reason is that we have to use the AspectJ compiler (ajc, an extension of the java compiler) to weave all classes that are affected by an aspect.</span> That's why, as I mention before, we need to add some extra configuration to our gradle build task to make it work.</p>
  
<p class="justify">This is how our <span class="boldtext">build.gradle</span> looks like:</p>

```groovy
import com.android.build.gradle.LibraryPlugin
import org.aspectj.bridge.IMessage
import org.aspectj.bridge.MessageHandler
import org.aspectj.tools.ajc.Main

buildscript {
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:0.12.+'
    classpath 'org.aspectj:aspectjtools:1.8.1'
  }
}

apply plugin: 'android-library'

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.aspectj:aspectjrt:1.8.1'
}

android {
  compileSdkVersion 19
  buildToolsVersion '19.1.0'

  lintOptions {
    abortOnError false
  }
}

android.libraryVariants.all { variant ->
  LibraryPlugin plugin = project.plugins.getPlugin(LibraryPlugin)
  JavaCompile javaCompile = variant.javaCompile
  javaCompile.doLast {
    String[] args = ["-showWeaveInfo",
                     "-1.5",
                     "-inpath", javaCompile.destinationDir.toString(),
                     "-aspectpath", javaCompile.classpath.asPath,
                     "-d", javaCompile.destinationDir.toString(),
                     "-classpath", javaCompile.classpath.asPath,
                     "-bootclasspath", plugin.project.android.bootClasspath.join(
        File.pathSeparator)]

    MessageHandler handler = new MessageHandler(true);
    new Main().run(args, handler)

    def log = project.logger
    for (IMessage message : handler.getMessages(null, true)) {
      switch (message.getKind()) {
        case IMessage.ABORT:
        case IMessage.ERROR:
        case IMessage.FAIL:
          log.error message.message, message.thrown
          break;
        case IMessage.WARNING:
        case IMessage.INFO:
          log.info message.message, message.thrown
          break;
        case IMessage.DEBUG:
          log.debug message.message, message.thrown
          break;
      }
    }
  }
}
```

## Our test method

<p class="justify">Let's use our cool aspect annotation by adding it to a test method. <span class="boldtext">I have created a method inside the main activity for testing purpose.</span> Let's have a look at it:</p>

```java
@DebugTrace
  private void testAnnotatedMethod() {
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
```

## Executing our application

<p class="justify">We build and install our app on an android device/emulator by executing the gradle command:</p>

```
gradlew clean build installDebug
```

<p class="justify">If we open the logcat and execute our sample, we will see a debug log with:</p>

```
Gintonic --> testAnnotatedMethod --> [10ms]
```

<p class="justify"><span class="boldtext">Our first android application using AOP worked!</span></p>
  
<p class="justify">You can use the <a href="https://play.google.com/store/apps/details?id=jp.itplus.android.dex.dump&hl=en" target="_blank">Dex Dump</a> android application (from your phone), or any any other reverse engineering tool for decompiling the apk and see the source code generated and injected.</p>

## Recap

So to recap and summarize:

  * <span class="boldtext">We have had a taste of Aspect Oriented programming paradigm.</span>
  * <span class="boldtext">Code Injection becomes a very important part of this approach (AOP).</span>
  * <span class="boldtext">AspectJ is a very powerful and easy to use tool for source code weaving in Android applications.</span>
  * <span class="boldtext">We have created a working example using AOP capabilities.</span>

## Conclusion

<p class="justify"><span class="boldtext">Aspect Oriented Programming is very powerful.</span> Using it the right way, you can avoid duplicating a lot of code when you have <span class="boldtext">"cross-cutting concerns"</span> in your Android apps, like performance monitoring, as we have seen in our example. I do encourage you to give it a try, you will find it very useful.</p>
  
<p class="justify"><span class="boldtext">I hope you like the article, the purpose of it was to share what I've learnt so far, so feel free to comment and give feedback, or even better, fork the code and play a bit with it.</span></p>
  
<p class="justify">I'm sure we can add very interesting stuff to our AOP module in the sample app. Ideas are very welcome ;).</p>

## Source Code

<p class="justify">You can check 2 examples here, the first one uses <span class="boldtext">AspectJ</span> and the second one uses a <span class="boldtext">Dynamic Proxy</span> approach:</p>

  * <a href="https://github.com/android10/Android-AOPExample" target="_blank">https://github.com/android10/Android-AOPExample</a>. 
  * <a href="https://github.com/android10/DynamicProxy_Java_Sample" target="_blank">https://github.com/android10/DynamicProxy_Java_Sample</a>

## Resources

  * <a href="http://en.wikipedia.org/wiki/Aspect-oriented_programming" target="_blank">Aspect-oriented programming.</a>
  * <a href="http://en.wikipedia.org/wiki/Aspect-oriented_software_development" target="_blank">Aspect-oriented software development.</a>
  * <a href="http://www.javacodegeeks.com/2011/09/practical-introduction-into-code.html" target="_blank">Practical Introduction into Code Injection with AspectJ, Javassist, and Java Proxy.</a>
  * <a href="http://java.dzone.com/articles/implementing-build-time" target="_blank">Implementing Build-time Bytecode Instrumentation With Javassist.</a>
  * <a href="http://www.eclipse.org/aspectj/doc/released/faq.php" target="_blank">Frequently Asked Questions about AspectJ.</a>
  * <a href="http://blog.espenberntsen.net/2010/03/20/aspectj-cheat-sheet/" target="_blank">AspectJ Cheat Sheet.</a>