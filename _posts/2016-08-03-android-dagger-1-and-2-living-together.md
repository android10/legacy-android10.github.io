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
<p style="text-align: justify;">
  In this article I will give a quick explanation on <span style="color: #333399;"><strong>how we can have Dagger 1 and 2 siting and working together</strong></span>. <span style="text-decoration: underline;">Let me just clear this up:</span> I will not talk about the benefits of neither dependency injection nor Dagger, since <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">I have done it in the past</a>. Also, the title of this post may sound a little bit <strong><span style="color: #333399;">misleading</span></strong> up front, but the idea is that hopefully by the end of it, you will understand its <strong><span style="color: #333399;">reason</span></strong>.
</p>

### Why?

<p style="text-align: justify;">
  This is the first question that comes to my mind at the moment of this writing. Well&#8230; let me give you some more context here: <strong><span style="color: #333399;">you have a big android/java codebase using Dagger 1 and want to reorganize your dependency injection approach and migrate to Dagger 2.</span></strong>
</p>

<p style="text-align: justify;">
  You have a couple of options:
</p>

<li style="text-align: justify;">
  <strong><span style="color: #333399;">Do it in one shot</span></strong>, for instance, send a big PR (the team is not going to be very happy when reviewing it and might lead to a lot of conflicts with existing development branches and features).
</li>
<li style="text-align: justify;">
  <span style="color: #333399;"><strong>Do it gradually in small pieces</strong></span> (<span style="text-decoration: underline;">divide and conquer:</span> always break down a big problem into smaller ones, specially in <span style="text-decoration: underline;">big codebases with many people contributing to it</span>.).
</li>

<img class="aligncenter size-full wp-image-468" src="http://fernandocejas.com/wp-content/uploads/2016/08/sc_lines_of_code.png" alt="sc_lines_of_code" width="968" height="84" srcset="http://fernandocejas.com/wp-content/uploads/2016/08/sc_lines_of_code.png 968w, http://fernandocejas.com/wp-content/uploads/2016/08/sc_lines_of_code-300x26.png 300w, http://fernandocejas.com/wp-content/uploads/2016/08/sc_lines_of_code-768x67.png 768w" sizes="(max-width: 968px) 100vw, 968px" />

<p style="text-align: justify;">
  Just for the record, the picture reflects our <strong><span style="color: #333399;">SoundCloud Listeners Android app main module which has 240k lines of code approximately</span></strong> (without counting internally developed libraries) and at least <strong><span style="color: #333399;">20 contributors</span></strong> to the main codebase.
</p>

### Solving the puzzle

<p style="text-align: justify;">
  <strong><span style="color: #333399;">When trying to use both Dagger versions together, you might run into different situations like classpath clashes and conflicts or transitive dependencies issues</span></strong>. So, in order to avoid them, we have to somehow <span style="color: #333399;"><strong>relocate Dagger 2 packages:</strong></span> this sounds scary but don&#8217;t give up and continue reading, I promise there is light at the end of the tunnel.
</p>

With that being said, we are going to use a gradle plugin called <span style="color: #333399;"><strong>&#8220;Shadow&#8221;</strong></span> created by <a href="https://twitter.com/johnrengelman" target="_blank">John Engelman</a>.
  
Basically the idea is to use the shadow functionality to achieve our goal, as it is described in the <a href="http://imperceptiblethoughts.com/shadow/" target="_blank">plugin official documentation</a>:

<p style="text-align: justify;">
  <strong><span style="color: #333399;">&#8220;Dependency bundling and relocation is the main use case for library authors. The goal of a bundled library is to create a pre-packaged dependency for other libraries or applications to utilize. Often in these scenarios, a library may contain a dependency that a downstream library or application also uses. In some cases, different versions of this common dependency can cause an issue in either the upstream library or the downstream application. These issues often manifest themselves as binary incompatibilities in either the library or application code. By utilizing Shadow’s ability to relocate the package names for dependencies, a library author can ensure that the library’s dependencies will not conflict with the same dependency being declared by the downstream application.&#8221;</span></strong>
</p>

### Putting all pieces together

<p style="text-align: justify;">
  <span style="text-decoration: underline;">Now we are ready to go</span>: given our android application project, the first step is to <strong><span style="color: #333399;">add 2 plain java library modules</span></strong> which are going to represent both dagger 2 main library and compiler, as you can see in the following picture:
</p>

<img class="aligncenter size-medium wp-image-470" src="http://fernandocejas.com/wp-content/uploads/2016/08/two-dagger-project-300x156.png" alt="two-dagger-project" width="300" height="156" srcset="http://fernandocejas.com/wp-content/uploads/2016/08/two-dagger-project-300x156.png 300w, http://fernandocejas.com/wp-content/uploads/2016/08/two-dagger-project.png 438w" sizes="(max-width: 300px) 100vw, 300px" />

<p style="text-align: justify;">
  <span style="text-decoration: underline;">The idea is simple:</span> <strong><span style="color: #333399;">we relocate Dagger 2 packages</span></strong> and you can see the magic happening if you sneak into the <strong><span style="color: #333399;">build.gradle</span></strong> file (<span style="text-decoration: underline;">shadowJar</span> configuration block) of any of the recently added projects:
</p>

<pre class="font-size:13 lang:java decode:true" title="build.gradle">shadowJar {
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
}</pre>

<p style="text-align: justify;">
  <span style="text-decoration: underline;">The script talks by itself here</span>, and now we have our <span style="color: #333399;"><strong>&#8220;shadowed&#8221;</strong></span> Dagger 2 version ready to be used in our project. <strong><span style="color: #333399;">Something to keep in mind is that in order to differentiate what is being injected by Dagger 1 and Dagger 2 we will have to use different annotations</span></strong>: this is necessary to avoid clashes. This will also bring upsides and downsides, but continue reading, we will get back to it later on.
</p>

<p style="text-align: justify;">
  Now it is time to setup our main application dependencies:
</p>

<pre class="font-size:13 lang:java decode:true " title="build.gradle">dependencies {
  //Dagger 1
  apt 'com.squareup.dagger:dagger-compiler:1.2.2'
  compile 'com.squareup.dagger:dagger:1.2.2'

  //Dagger 2
  apt project(path: ':two-daggers-compiler', transitive: false)
  compile project(path: ':two-daggers-library', transitive: false)

  testCompile 'junit:junit:4.12'
}</pre>

<p style="text-align: justify;">
  <span style="color: #333399;"><strong>Dagger 1 remains the same but we have our new friend coming from compiled &#8220;relocated&#8221; projects.</strong></span> This should work out of the box.
</p>

### Alternative setup

<p style="text-align: justify;">
  If you do not want to deal with adding 2 extra library components to your project, another way is<strong><span style="color: #333399;"> to use the 2 jars generated</span></strong> by <a href="https://github.com/android10/two-daggers" target="_blank">my sample project</a>, place them into <span style="color: #333399;"><strong>/libs or any other folder</strong></span> (or you can also use a <span style="text-decoration: underline;">Repository Manager</span> like a <span style="text-decoration: underline;">Nexus Server</span> for example) and setup the dependencies like this:
</p>

<img class="aligncenter wp-image-473 size-medium" src="http://fernandocejas.com/wp-content/uploads/2016/08/two-daggers-jars-300x160.png" width="300" height="160" srcset="http://fernandocejas.com/wp-content/uploads/2016/08/two-daggers-jars-300x160.png 300w, http://fernandocejas.com/wp-content/uploads/2016/08/two-daggers-jars.png 502w" sizes="(max-width: 300px) 100vw, 300px" />

<pre class="font-size:13 lang:java decode:true " title="build.gradle">dependencies {
  ...

  /* Only for Dagger 1 to 2 migration */
  apt files('dagger/two-daggers-compiler.jar')

  /* Only for Dagger 1 to 2 migration */
  compile files('dagger/two-daggers.jar')

  ...
}</pre>

### Migration process

<p style="text-align: justify;">
  Before you jump on this journey, <span style="text-decoration: underline;">here is a little reminder</span>: <strong><span style="color: #333399;">this is worth if you really have a big codebase and you want to do the migration gradually, otherwise always keep it simple and neither do over engineering nor reinvent the wheel.</span></strong> You have been warned ;).
</p>

<p style="text-align: justify;">
  Now that you decided to hopefully continue, here is an example of a class that can be injected by <strong><span style="color: #333399;">both Daggers</span></strong> (<a href="https://github.com/android10/two-daggers" target="_blank">more code in my sample project on github</a>):
</p>

<pre class="font-size:13 lang:java decode:true " title="String Generator">@Singleton
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
}</pre>

<p style="text-align: justify;">
  <span style="text-decoration: underline;">By looking at this class</span> you can tell that it is a <span style="text-decoration: underline;">Singleton</span> (<strong><span style="color: #333399;">@Singleton</span></strong> annotation for Dagger 1 and <strong><span style="color: #333399;">@Singleton2</span></strong> annotation for Dagger 2). In this case we are not saving any global state (which is good) but there might be cases where you cannot avoid situation and <strong><span style="color: #333399;">might run into weird and unexpected behavior due to the fact that you have 2 different instances of the same class being injected, sitting in different dependency graphs. For instance, this is not longer a Singleton and it is definitely dangerous, so make sure you to take responsibility on this and your tests cover all these cases.</span></strong>
</p>

<p style="text-align: justify;">
  When the migration is over you can remove Dagger 1 and setup Dagger 2 dependency as you would do with any other one. Afterwards, you will have to get rid of all Dagger 1 related annotations plus the ones that Dagger 2 do not understand anymore and replace them:
</p>

  * <span style="color: #333399;"><strong>@Inject2</strong></span> &#8230;with&#8230; **<span style="color: #333399;">@Inject</span>**
  * <span style="color: #333399;"><strong>@Singleton2</strong></span> &#8230;with&#8230; **<span style="color: #333399;">@Singleton</span>**
  * <span style="color: #333399;"><strong>@Component2</strong></span> &#8230;with&#8230; **<span style="color: #333399;">@Component</span>**
  * <span style="color: #333399;"><strong>@Provider2</strong></span> &#8230;with&#8230; <span style="color: #333399;"><strong>@Provider</strong></span>
  * <span style="color: #333399;"><strong>@Qualifier2</strong></span> &#8230;with&#8230; <span style="color: #333399;"><strong>@Qualifier</strong></span>
  * <span style="color: #333399;"><strong>@Scope2</strong></span> &#8230;with&#8230; <span style="color: #333399;"><strong>@Scope</strong></span>
  * <span style="color: #333399;"><strong>@Lazy2</strong></span> &#8230;with&#8230; <span style="color: #333399;"><strong>@Lazy</strong></span>

<p style="text-align: justify;">
  Hopefully at this point you will have everything <span style="text-decoration: underline;">up and running</span> and the job done little by little <strong><span style="color: #333399;">without much suffering and pain</span></strong> so happy migration!
</p>

### Conclusion

<p style="text-align: justify;">
  <strong><span style="color: #333399;">In this article we have learned and prepared the terrain to make Dagger 1 and 2 live together in a friendly way.</span></strong> Keep in mind that this solutions <strong><span style="text-decoration: underline; color: #333399;">does not only apply to Dagger</span></strong> but to <strong><span style="text-decoration: underline;"><span style="color: #333399; text-decoration: underline;">any other similar potential migration process you might face in the future.</span></span></strong>
</p>

<p style="text-align: justify;">
  Finally I hope you find this article useful, and as usual any feedback is very welcome and important. You can find me on <strong><span style="color: #333399;">twitter</span></strong>: <a href="https://twitter.com/fernando_cejas" target="_blank">@fernando_cejas</a>.
</p>

### References

  * [Sample code on github](https://github.com/android10/two-daggers)
  * <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">Tasting Dagger 2 on Android</a>
  * <a href="https://www.youtube.com/watch?v=dfyEzLFS-uA" target="_blank">Sharper Better Faster Dagger</a>
  * <a href="https://www.youtube.com/watch?v=3B7F7emCc64" target="_blank">The Journey of Android Engineers: A Tale of Two Daggers</a>
  * <a href="https://github.com/johnrengelman/shadow" target="_blank">Shadow Gradle Plugin</a>
  * <a href="http://imperceptiblethoughts.com/shadow/" target="_blank">Shadows Plugin Documentation</a>