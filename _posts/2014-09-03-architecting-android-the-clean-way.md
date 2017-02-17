---
id: 4
title: 'Architecting Android...The clean way?'
date: 2014-09-03T01:45:54+00:00
author: Fernando Cejas
layout: post
permalink: /2014/09/03/architecting-android-the-clean-way/
categories:
  - Android
  - Development
  - Software Architecture
  - Testing
tags:
  - android
  - androiddev
  - architecture
  - clean
  - clean architecture
  - developer
  - developers
  - development
  - model view presenter
  - mvp
  - pattern
  - patterns
  - programming
  - repository
  - repository pattern
  - strategy pattern
---
<p class="justify">Over the last months and after having a few android discussions at <a href="http://corporate.tuenti.com/en/dev/blog" target="_blank">Tuenti</a> with colleagues like <a href="https://twitter.com/pedro_g_s" target="_blank">@pedro_g_s</a> and <a href="https://twitter.com/flipper83" target="_blank">@flipper83</a>, I have decided that was a good time to write an article about <span class="boldtext">architecting android applications.</span></p>
  
<p class="justify">The purpose of it is to show you a little approach I had in mind in the last few months plus all the stuff I have learnt from investigating and implementing it.</p>

## Getting Started

<p class="justify"><span class="boldtext">We know that writing quality software is hard and complex:</span> It is not only about satisfying requirements, also should be robust, maintainable, testable, and flexible enough to adapt to growth and change. This is where <span class="boldtext">"the clean architecture"</span> comes up and could be a good approach for using when developing any software application.</p>
  
<p class="justify">The idea is simple: <span class="boldtext">clean architecture</span> stands for a group of practices that produce systems that are:</p>

  * <span class="boldtext">Independent of Frameworks.</span>
  * <span class="boldtext">Testable.</span>
  * <span class="boldtext">Independent of UI.</span>
  * <span class="boldtext">Independent of Database.</span>
  * <span class="boldtext">Independent of any external agency.</span>

<img class="aligncenter wp-image-208 size-full" src="/assets/migrated/clean_architecture1.png" alt="" width="647" height="440" srcset="/assets/migrated/clean_architecture1.png 647w, /assets/migrated/clean_architecture1-300x204.png 300w" sizes="(max-width: 647px) 100vw, 647px" />

<p class="justify">It is not a must to use only 4 circles (as you can see in the picture), because they are only schematic but you should take into consideration the <span class="boldtext">Dependency Rule: source code dependencies can only point inwards and nothing in an inner circle can know anything at all about something in an outer circle.</span></p>

<p class="justify">Here is some vocabulary that is relevant for getting familiar and understanding this approach in a better way:</p>

  * <span class="boldtext">Entities:</span> These are the business objects of the application.
  * <span class="boldtext">Use Cases:</span> These use cases orchestrate the flow of data to and from the entities. Are also called Interactors.
  * <span class="boldtext">Interface Adapters:</span> This set of adapters convert data from the format most convenient for the use cases and entities. Presenters and Controllers belong here.
  * <span class="boldtext">Frameworks and Drivers:</span> This is where all the details go: UI, tools, frameworks, etc.

<p class="justify">For a better and more extensive explanation, refer to <a href="http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html" target="_blank">this article</a> or <a href="http://vimeo.com/43612849" target="_blank">this video</a>.</p>

## Our Scenario

<p class="justify"><span class="boldtext">I will start with a simple scenario to get things going:</span> simply create an small app that shows a list of friends or users retrieved from the cloud and, when clicking any of them, a new screen will be opened and for instance, show more details for that user. <span class="underlinetext">Here is a quick video:</span></p>

<center><iframe width="640" height="480" src="//www.youtube.com/embed/XSjV4sG3ni0" frameborder="0" allowfullscreen="allowfullscreen"></iframe></center>

## Android Architecture

<p class="justify">The purpose is the <span class="boldtext">separation of concerns</span> by keeping the business rules not knowing anything at all about the outside world, thus, they can can be tested without any dependency to any external element.</p>
  
<p class="justify">To achieve this, <span class="boldtext">my proposal is about breaking up the project into 3 different layers,</span> in which each one has its own purpose and works separately from the others.</p>
  
<p class="justify"><span class="boldtext">It is worth mentioning that each layer uses its own data model so this independence can be reached </span> (you will see in code that a data mapper is needed in order to accomplish data transformation, a price to be paid if you do not want to cross the use of your models over the entire application).</p>
  
<p class="justify"><span class="boldtext">Here is an schema so you can see how it looks like:</span></p>

<img class="aligncenter size-full wp-image-206" src="/assets/migrated/clean_architecture_android.png" alt="clean_architecture_android" width="673" height="292" srcset="/assets/migrated/clean_architecture_android.png 673w, /assets/migrated/clean_architecture_android-300x130.png 300w" sizes="(max-width: 673px) 100vw, 673px" />

<p class="justify"><span class="boldtext">NOTE:</span> I did not use any external library (except gson for parsing json data and junit, mockito, robolectric and espresso for testing). <span class="boldtext">The reason is that it makes the example clearer.</span> Anyway do not hesitate to add ORMs for storing disk data or any dependency injection framework or whatever tool or library you are familiar with, that could make your life easier. <span class="boldtext">(Remember that reinventing the wheel is not a good practice).</span></p>

## Presentation Layer

<p class="justify">Is here, where the logic related with views and animations happens. It uses no more than a <span class="boldtext">Model View Presenter</span> (<a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter" target="_blank">MVP</a> from now on), but you can use any other pattern like MVC or MVVM. I will not get into details on it, but here <span class="boldtext">fragments and activities are only views,</span> there is no logic inside them other than UI logic, and this is where all the rendering stuff takes place.</p>
  
<p class="justify"><span class="boldtext">Presenters</span> in this layer are composed with <span class="boldtext">interactors (use cases)</span> that perform the job in a <span class="boldtext">new thread outside the main android UI thread</span>, and come back using a callback with the data that will be rendered in the view.</p>

<img class="aligncenter size-full wp-image-210" src="/assets/migrated/clean_architecture_mvp.png" alt="clean_architecture_mvp" width="406" height="238" srcset="/assets/migrated/clean_architecture_mvp.png 406w, /assets/migrated/clean_architecture_mvp-300x175.png 300w" sizes="(max-width: 406px) 100vw, 406px" />

<p class="justify">If you want a cool example about <a href="https://github.com/pedrovgs/EffectiveAndroidUI/" target="_blank">Effective Android UI</a> that uses MVP and MVVM, take a look at what my friend <span class="underlinetext">Pedro Gómez has done.</span></p>

## Domain Layer

<p class="justify"><span class="boldtext">Business rules here: all the logic happens in this layer.</span> Regarding the android project, you will see all the interactors (use cases) implementations here as well.</p>
  
<p class="justify"><span class="boldtext">This layer is a pure java module without any android dependencies.</span> All the external components use interfaces when connecting to the business objects.</p>

<img class="aligncenter size-full wp-image-212" src="/assets/migrated/clean_architecture_domain.png" alt="clean_architecture_domain" width="297" height="280" />

## Data Layer

<p class="justify"><span class="boldtext">All data needed for the application comes from this layer through a UserRepository implementation (the interface is in the domain layer) that uses a <a href="http://martinfowler.com/eaaCatalog/repository.html" target="_blank">Repository Pattern</a> with a strategy that, through a factory, picks different data sources depending on certain conditions.</span></p>
  
<p class="justify">For instance, when getting a user by id, the disk cache data source will be selected if the user already exists in cache, otherwise the cloud will be queried to retrieve the data and later save it to the disk cache.</p>
  
<p class="justify"><span class="boldtext">The idea behind all this is that the data origin is transparent for the client,</span> which does not care if the data is coming from memory, disk or the cloud, the only truth is that the data will arrive and will be got.</p>

<img class="aligncenter size-full wp-image-214" src="/assets/migrated/clean_architecture_data.png" alt="clean_architecture_data" width="698" height="385" srcset="/assets/migrated/clean_architecture_data.png 698w, /assets/migrated/clean_architecture_data-300x165.png 300w" sizes="(max-width: 698px) 100vw, 698px" />

<p class="justify"><span class="boldtext">NOTE:</span> In terms of code I have implemented a very simple and primitive disk cache using the file system and android preferences, it was for learning purpose. Remember again that you <span class="boldtext">SHOULD NOT REINVENT THE WHEEL</span> if there are existing libraries that perform these jobs in a better way.</p>

## Error Handling

<p class="justify">This is always a topic for discussion and could be great if you share your solutions here.</p>
  
<p class="justify"><span class="boldtext">My strategy was to use callbacks,</span> thus, if something happens in the data repository for example, the callback has 2 methods <span class="boldtext">onResponse()</span> and <span class="boldtext">onError().</span> The last one encapsulates exceptions in a wrapper class called <span class="boldtext">"ErrorBundle":</span> This approach brings some difficulties because there is a chains of callbacks one after the other until the error goes to the presentation layer to be rendered. Code readability could be a bit compromised.</p>
  
<p class="justify">On the other side, I could have implemented an event bus system that throws events if something wrong happens but this kind of solution is like using a <a href="http://www.drdobbs.com/jvm/programming-with-reason-why-is-goto-bad/228200966" target="_blank">GOTO</a>, and, <span class="boldtext">in my opinion, sometimes you can get lost when you're subscribed to several events if you do not control that closely.</span></p>

## Testing

<p class="justify">Regarding testing, I opted for several solutions depending on the layer:</p>

  * <span class="boldtext">Presentation Layer:</span> used android instrumentation and espresso for integration and functional testing.
  * <span class="boldtext">Domain Layer:</span> JUnit plus mockito for unit tests was used here.
  * <span class="boldtext">Data Layer:</span> Robolectric (since this layer has android dependencies) plus junit plus mockito for integration and unit tests.

## Show me the code

<p class="justify"><span class="boldtext">I know that you may be wondering where is the code, right?</span> Well <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">here is the github link</a> where you will find what I have done. About the folder structure, something to mention, is that the different layers are represented using modules:</p>

  * <span class="boldtext">presentation:</span> It is an android module that represents the presentation layer.
  * <span class="boldtext">domain:</span> A java module without android dependencies.
  * <span class="boldtext">data:</span> An android module from where all the data is retrieved.
  * <span class="boldtext">data-test:</span> Tests for the data layer. Due to some limitations when using Robolectric I had to use it in a separate java module.

## Conclusion

<p class="justify">As Uncle Bob says, <span class="boldtext">"Architecture is About Intent, not Frameworks"</span> and I totally agree with this statement. Of course there are a lot of different ways of doing things (different implementations) and I'm pretty sure that you (like me) face a lot of challenges every day, but <span class="boldtext">by using this technique,</span> you make sure that your application will be:</p>

  * <span class="boldtext">Easy to maintain.</span>
  * <span class="boldtext">Easy to test.</span>
  * <span class="boldtext">Very cohesive.</span>
  * <span class="boldtext">Decoupled.</span>

<p class="justify"><span class="boldtext">As a conclusion I strongly recommend you give it a try and see and share your results and experiences,</span> as well as any other approach you’ve found that works better: we do know that <span class="boldtext">continuous improvement</span> is always a very good and positive thing.</p>
  
<p class="justify">I hope you have found this article useful and, as always, any feedback is very welcome.</p>

## Source code

  * <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Clean architecture github repository &#8211; master branch</a>
  * <a href="https://github.com/android10/Android-CleanArchitecture/releases" target="_blank">Clean architecture github repository &#8211; releases</a>

## Further reading:

  * <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android..the evolution</a>
  * <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">Tasting Dagger 2 on Android</a>
  * <a href="https://speakerdeck.com/android10/the-mayans-lost-guide-to-rxjava-on-android" target="_blank">The Mayans Lost Guide to RxJava on Android</a>
  * <a href="https://speakerdeck.com/android10/it-is-about-philosophy-culture-of-a-good-programmer" target="_blank">It is about philosophy: Culture of a good programmer</a>

## Links and Resources

  * <a href="http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html" target="_blank">The clean architecture by Uncle Bob</a>
  * <a href="http://www.infoq.com/news/2013/07/architecture_intent_frameworks" target="_blank">Architecture is about Intent, not Frameworks</a>
  * <a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter" target="_blank">Model View Presenter</a>
  * <a href="http://martinfowler.com/eaaCatalog/repository.html" target="_blank">Repository Pattern by Martin Fowler</a>
  * <a href="http://www.slideshare.net/PedroVicenteGmezSnch/" target="_blank">Android Design Patterns Presentation</a>