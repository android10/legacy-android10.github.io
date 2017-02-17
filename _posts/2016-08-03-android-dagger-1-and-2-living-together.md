---
id: 10
title: 'Android: Dagger 1 and 2 living together'
date: 2016-08-03T20:10:03+00:00
author: Fernando Cejas
layout: post
permalink: /2016/08/03/android-dagger-1-and-2-living-together/
categories:
  - Android
  - Development
  - Java
tags:
  - android
  - androiddev
  - code injection
  - dagger
  - dagger 2
  - dagger2
  - dependency injection
  - dependency inversion
  - developer
  - developers
  - development
  - java
  - programming
---
<p class="justify">In this article I will give a quick explanation on <span class="boldtext">how we can have Dagger 1 and 2 siting and working together</span>. <span class="underlinetext">Let me just clear this up:</span> I will not talk about the benefits of neither dependency injection nor Dagger, since <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">I have done it in the past</a>. Also, the title of this post may sound a little bit <span class="boldtext">misleading</span> up front, but the idea is that hopefully by the end of it, you will understand its <span class="boldtext">reason</span>.</p>

## Why?

<p class="justify">This is the first question that comes to my mind at the moment of this writing. Well... let me give you some more context here: <span class="boldtext">you have a big android/java codebase using Dagger 1 and want to reorganize your dependency injection approach and migrate to Dagger 2.</span></p>

<p class="justify">You have a couple of options:</p>

  * <p class="justify"><span class="boldtext">Do it in one shot</span>, for instance, send a big PR (the team is not going to be very happy when reviewing it and might lead to a lot of conflicts with existing development branches and features).</p>
  * <p class="justify"><span class="boldtext">Do it gradually in small pieces</span> (<span class="underlinetext">divide and conquer:</span> always break down a big problem into smaller ones, specially in <span class="underlinetext">big codebases with many people contributing to it</span>.).</p>

<img class="aligncenter size-full wp-image-468" src="/assets/migrated/sc_lines_of_code.png" alt="sc_lines_of_code" width="968" height="84" srcset="/assets/migrated/sc_lines_of_code.png 968w, /assets/migrated/sc_lines_of_code-300x26.png 300w, /assets/migrated/sc_lines_of_code-768x67.png 768w" sizes="(max-width: 968px) 100vw, 968px" />

<p class="justify">Just for the record, the picture reflects our <span class="boldtext">SoundCloud Listeners Android app main module which has 240k lines of code approximately</span> (without counting internally developed libraries) and at least <span class="boldtext">20 contributors</span> to the main codebase.</p>

## Solving the puzzle

<p class="justify"><span class="boldtext">When trying to use both Dagger versions together, you might run into different situations like classpath clashes and conflicts or transitive dependencies issues</span>. So, in order to avoid them, we have to somehow <span class="boldtext">relocate Dagger 2 packages:</span> this sounds scary but do not give up and continue reading, I promise there is light at the end of the tunnel.</p>

With that being said, we are going to use a gradle plugin called <span class="boldtext">"Shadow"</span> created by <a href="https://twitter.com/johnrengelman" target="_blank">John Engelman</a>.
  
Basically the idea is to use the shadow functionality to achieve our goal, as it is described in the <a href="http://imperceptiblethoughts.com/shadow/" target="_blank">plugin official documentation</a>:

<p class="justify"><span class="boldtext">"Dependency bundling and relocation is the main use case for library authors. The goal of a bundled library is to create a pre-packaged dependency for other libraries or applications to utilize. Often in these scenarios, a library may contain a dependency that a downstream library or application also uses. In some cases, different versions of this common dependency can cause an issue in either the upstream library or the downstream application. These issues often manifest themselves as binary incompatibilities in either the library or application code. By utilizing Shadow’s ability to relocate the package names for dependencies, a library author can ensure that the library’s dependencies will not conflict with the same dependency being declared by the downstream application."</span></p>

## Putting all pieces together

<p class="justify"><span class="underlinetext">Now we are ready to go</span>: given our android application project, the first step is to <span class="boldtext">add 2 plain java library modules</span> which are going to represent both dagger 2 main library and compiler, as you can see in the following picture:</p>

<img class="aligncenter size-medium wp-image-470" src="/assets/migrated/two-dagger-project-300x156.png" alt="two-dagger-project" width="300" height="156" srcset="/assets/migrated/two-dagger-project-300x156.png 300w, /assets/migrated/two-dagger-project.png 438w" sizes="(max-width: 300px) 100vw, 300px" />

<p class="justify"><span class="underlinetext">The idea is simple:</span> <span class="boldtext">we relocate Dagger 2 packages</span> and you can see the magic happening if you sneak into the <span class="boldtext">build.gradle</span> file (<span class="underlinetext">shadowJar</span> configuration block) of any of the recently added projects:</p>

```groovy
shadowJar {
  relocate 'javax.inject.Inject', 'javax.inject.Inject2'
  relocate 'javax.inject.Named', 'javax.inject.Named2'
  relocate 'javax.inject.Provider', 'javax.inject.Provider2'
  relocate 'javax.inject.Qualifier', 'javax.inject.Qualifier2'
  relocate 'javax.inject.Scope', 'javax.inject.Scope2'
  relocate 'javax.inject.Singleton', 'javax.inject.Singleton2'
  relocate 'dagger.Lazy', 'dagger.Lazy2'
  relocate 'dagger.MembersInjector', 'dagger.MembersInjector2'
  relocate 'dagger.Module', 'dagger.Module2'
  relocate 'dagger.Provides', 'dagger.Provides2'
  relocate 'dagger.Provides$Type', 'dagger.Provides$Type2'
  relocate 'dagger.internal', 'dagger.internal2'
  relocate 'dagger.producers', 'dagger.producers2'
}
```

<p class="justify"><span class="underlinetext">The script talks by itself here</span>, and now we have our <span class="boldtext">"shadowed"</span> Dagger 2 version ready to be used in our project. <span class="boldtext">Something to keep in mind is that in order to differentiate what is being injected by Dagger 1 and Dagger 2 we will have to use different annotations</span>: this is necessary to avoid clashes. This will also bring upsides and downsides, but continue reading, we will get back to it later on.</p>

<p class="justify">Now it is time to setup our main application dependencies:</p>

```groovy
dependencies {
  //Dagger 1
  apt 'com.squareup.dagger:dagger-compiler:1.2.2'
  compile 'com.squareup.dagger:dagger:1.2.2'

  //Dagger 2
  apt project(path: ':two-daggers-compiler', transitive: false)
  compile project(path: ':two-daggers-library', transitive: false)

  testCompile 'junit:junit:4.12'
}
```

<p class="justify"><span class="boldtext">Dagger 1 remains the same but we have our new friend coming from compiled "relocated" projects.</span> This should work out of the box.</p>

## Alternative setup

<p class="justify">If you do not want to deal with adding 2 extra library components to your project, another way is<span class="boldtext"> to use the 2 jars generated</span> by <a href="https://github.com/android10/two-daggers" target="_blank">my sample project</a>, place them into <span class="boldtext">/libs or any other folder</span> (or you can also use a <span class="underlinetext">Repository Manager</span> like a <span class="underlinetext">Nexus Server</span> for example) and setup the dependencies like this:</p>

<img class="aligncenter wp-image-473 size-medium" src="/assets/migrated/two-daggers-jars-300x160.png" width="300" height="160" srcset="/assets/migrated/two-daggers-jars-300x160.png 300w, /assets/migrated/two-daggers-jars.png 502w" sizes="(max-width: 300px) 100vw, 300px" />

```groovy
dependencies {
  ...

  /* Only for Dagger 1 to 2 migration */
  apt files('dagger/two-daggers-compiler.jar')

  /* Only for Dagger 1 to 2 migration */
  compile files('dagger/two-daggers.jar')

  ...
}
```

## Migration process

<p class="justify">Before you jump on this journey, <span class="underlinetext">here is a little reminder</span>: <span class="boldtext">this is worth if you really have a big codebase and you want to do the migration gradually, otherwise always keep it simple and neither do over engineering nor reinvent the wheel.</span> You have been warned ;).</p>

<p class="justify">Now that you decided to hopefully continue, here is an example of a class that can be injected by <span class="boldtext">both Daggers</span> (<a href="https://github.com/android10/two-daggers" target="_blank">more code in my sample project on github</a>):</p>

```java
@Singleton
@Singleton2
public class StringGenerator {

  @Inject
  @Inject2
  /**
   * This class can be injected by both dagger 1 and 2
   */
  public StringGenerator() {
    //no op
  }

  public String randomValue() {
    //for learning purpose
    return "34290df320e2k0id";
  }
}
```

<p class="justify"><span class="underlinetext">By looking at this class</span> you can tell that it is a <span class="underlinetext">Singleton</span> (<span class="boldtext">@Singleton</span> annotation for Dagger 1 and <span class="boldtext">@Singleton2</span> annotation for Dagger 2). In this case we are not saving any global state (which is good) but there might be cases where you cannot avoid this situation and <span class="boldtext">might run into weird and unexpected behavior due to the fact that you have 2 different instances of the same class being injected, sitting in different dependency graphs. For instance, this is not longer a Singleton and it is definitely dangerous, so make sure you to take responsibility on this and your tests cover all these cases.</span></p>

<p class="justify">When the migration is over you can remove Dagger 1 and setup Dagger 2 dependency as you would do with any other one. Afterwards, you will have to get rid of all Dagger 1 related annotations plus the ones that Dagger 2 do not understand anymore and replace them:</p>

  * <span class="boldtext">@Inject2</span> ...with... <span class="boldtext">@Inject</span>
  * <span class="boldtext">@Singleton2</span> ...with... <span class="boldtext">@Singleton</span>
  * <span class="boldtext">@Component2</span> ...with... <span class="boldtext">@Component</span>
  * <span class="boldtext">@Provider2</span> ...with... <span class="boldtext">@Provider</span>
  * <span class="boldtext">@Qualifier2</span> ...with... <span class="boldtext">@Qualifier</span>
  * <span class="boldtext">@Scope2</span> ...with... <span class="boldtext">@Scope</span>
  * <span class="boldtext">@Lazy2</span> ...with... <span class="boldtext">@Lazy</span>

<p class="justify">Hopefully at this point you will have everything <span class="underlinetext">up and running</span> and the job done little by little <span class="boldtext">without much suffering and pain</span> so happy migration!</p>

## Conclusion

<p class="justify"><span class="boldtext">In this article we have learned and prepared the terrain to make Dagger 1 and 2 live together in a friendly way.</span> Keep in mind that this solutions <span class="underlinetext">does not only apply to Dagger</span> but to <span class="underlinetext">any other similar potential migration process you might face in the future.</span></p>

<p class="justify">Finally I hope you find this article useful, and as usual any feedback is very welcome and important. You can find me on <span class="boldtext">twitter</span>: <a href="https://twitter.com/fernando_cejas" target="_blank">@fernando_cejas</a>.</p>

## References

  * [Sample code on github](https://github.com/android10/two-daggers)
  * <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">Tasting Dagger 2 on Android</a>
  * <a href="https://www.youtube.com/watch?v=dfyEzLFS-uA" target="_blank">Sharper Better Faster Dagger</a>
  * <a href="https://www.youtube.com/watch?v=3B7F7emCc64" target="_blank">The Journey of Android Engineers: A Tale of Two Daggers</a>
  * <a href="https://github.com/johnrengelman/shadow" target="_blank">Shadow Gradle Plugin</a>
  * <a href="http://imperceptiblethoughts.com/shadow/" target="_blank">Shadows Plugin Documentation</a>