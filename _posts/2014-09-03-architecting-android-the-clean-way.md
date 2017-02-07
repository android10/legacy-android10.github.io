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
Over the last months and after having friendly discussions at <a href="http://corporate.tuenti.com/en/dev/blog" target="_blank">Tuenti</a> with colleagues like <a href="https://twitter.com/pedro_g_s" target="_blank">@pedro_g_s</a> and <a href="https://twitter.com/flipper83" target="_blank">@flipper83</a> (by the way 2 badass of android development), I have decided that was a good time to write an article about **<span style="color: #009933;">architecting android applications.</span>**
  
The purpose of it is to show you a little approach I had in mind in the last few months plus all the stuff I have learnt from investigating and implementing it.

### **Getting Started**

**<span style="color: #009933;">We know that writing quality software is hard and complex:</span>** It is not only about satisfying requirements, also should be robust, maintainable, testable, and flexible enough to adapt to growth and change. This is where **<span style="color: #009933;">&#8220;the clean architecture&#8221;</span>** comes up and could be a good approach for using when developing any software application.
  
The idea is simple: **<span style="color: #009933;">clean architecture</span>** stands for a group of practices that produce systems that are:

  * **<span style="color: #009933;">Independent of Frameworks.</span>**
  * **<span style="color: #009933;">Testable.</span>**
  * **<span style="color: #009933;">Independent of UI.</span>**
  * **<span style="color: #009933;">Independent of Database.</span>**
  * **<span style="color: #009933;">Independent of any external agency.</span>**

[<img class="aligncenter wp-image-208 size-full" src="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture1.png" alt="" width="647" height="440" srcset="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture1.png 647w, http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture1-300x204.png 300w" sizes="(max-width: 647px) 100vw, 647px" />](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture1.png)

It is not a must to use only 4 circles (as you can see in the picture), because they are only schematic but you should take into consideration the **<span style="color: #009933;">Dependency Rule:</span>** source code dependencies can only point inwards and nothing in an inner circle can know anything at all about something in an outer circle.

Here is some vocabulary that is relevant for getting familiar and understanding this approach in a better way:

  * **<span style="color: #009933;">Entities:</span>** These are the business objects of the application.
  * **<span style="color: #009933;">Use Cases:</span>** These use cases orchestrate the flow of data to and from the entities. Are also called Interactors.
  * **<span style="color: #009933;">Interface Adapters:</span>** This set of adapters convert data from the format most convenient for the use cases and entities. Presenters and Controllers belong here.
  * **<span style="color: #009933;">Frameworks and Drivers:</span>** This is where all the details go: UI, tools, frameworks, etc.

For a better and more extensive explanation, refer to <a href="http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html" target="_blank">this article</a> or <a href="http://vimeo.com/43612849" target="_blank">this video</a>.

### **Our Scenario**

I will start with a simple scenario to get things going: simply create an small app that shows a list of friends or users retrieved from the cloud and, when clicking any of them, a new screen will be opened showing more details of that user.
  
I leave you a video so you can have the big picture of what I&#8217;m talking about:

<center>
</center>

### **Android Architecture**

The objective is the **<span style="color: #009933;">separation of concerns</span>** by keeping the business rules not knowing anything at all about the outside world, thus, they can can be tested without any dependency to any external element.
  
To achieve this, **<span style="color: #009933;">my proposal is about breaking up the project into 3 different layers,</span>** in which each one has its own purpose and works separately from the others.
  
It is worth mentioning that each layer uses its own data model so this independence can be reached (you will see in code that a data mapper is needed in order to accomplish data transformation, a price to be paid if you do not want to cross the use of your models over the entire application).
  
Here is an schema so you can see how it looks like:

[<img class="aligncenter size-full wp-image-206" src="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_android.png" alt="clean_architecture_android" width="673" height="292" srcset="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_android.png 673w, http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_android-300x130.png 300w" sizes="(max-width: 673px) 100vw, 673px" />](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_android.png)

**<span style="color: #009933;">NOTE:</span>** I did not use any external library (except gson for parsing json data and junit, mockito, robolectric and espresso for testing). The reason was because it made the example a bit more clear. Anyway do not hesitate to add ORMs for storing disk data or any dependency injection framework or whatever tool or library you are familiar with, that could make your life easier. **<span style="color: #009933;">(Remember that reinventing the wheel is not a good practice).</span>**

#### **Presentation Layer**

Is here, where the logic related with views and animations happens. It uses no more than a **<span style="color: #009933;">Model View Presenter</span>** (<a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter" target="_blank">MVP</a> from now on), but you can use any other pattern like MVC or MVVM. I will not get into details on it, but here **<span style="color: #009933;">fragments and activities are only views,</span>** there is no logic inside them other than UI logic, and this is where all the rendering stuff takes place.
  
**<span style="color: #009933;">Presenters</span>** in this layer are composed with **<span style="color: #009933;">interactors (use cases)</span>** that perform the job in a new thread outside the android UI thread, and come back using a callback with the data that will be rendered in the view.

[<img class="aligncenter size-full wp-image-210" src="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_mvp.png" alt="clean_architecture_mvp" width="406" height="238" srcset="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_mvp.png 406w, http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_mvp-300x175.png 300w" sizes="(max-width: 406px) 100vw, 406px" />](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_mvp.png)

If you want a cool example about <a href="https://github.com/pedrovgs/EffectiveAndroidUI/" target="_blank">Effective Android UI</a> that uses MVP and MVVM, take a look at what my friend Pedro Gómez has done.

#### **Domain Layer**

**<span style="color: #009933;">Business rules here: all the logic happens in this layer.</span>** Regarding the android project, you will see all the interactors (use cases) implementations here as well.
  
**<span style="color: #009933;">This layer is a pure java module without any android dependencies.</span>** All the external components use interfaces when connecting to the business objects.

[<img class="aligncenter size-full wp-image-212" src="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_domain.png" alt="clean_architecture_domain" width="297" height="280" />](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_domain.png)

#### **Data Layer**

**<span style="color: #009933;">All data needed for the application comes from this layer through a UserRepository implementation (the interface is in the domain layer) that uses a <a href="http://martinfowler.com/eaaCatalog/repository.html" target="_blank">Repository Pattern</a> with a strategy that, through a factory, picks different data sources depending on certain conditions.</span>**
  
For instance, when getting a user by id, the disk cache data source will be selected if the user already exists in cache, otherwise the cloud will be queried to retrieve the data and later save it to the disk cache.
  
**<span style="color: #009933;">The idea behind all this is that the data origin is transparent for the client,</span>** which does not care if the data is coming from memory, disk or the cloud, the only truth is that the data will arrive and will be got.

[<img class="aligncenter size-full wp-image-214" src="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_data.png" alt="clean_architecture_data" width="698" height="385" srcset="http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_data.png 698w, http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_data-300x165.png 300w" sizes="(max-width: 698px) 100vw, 698px" />](http://fernandocejas.com/wp-content/uploads/2014/09/clean_architecture_data.png)

**<span style="color: #009933;">NOTE:</span>** In terms of code I have implemented a very simple and primitive disk cache using the file system and android preferences, it was for learning purpose. Remember again that you **<span style="color: #009933;">SHOULD NOT REINVENT THE WHEEL</span>** if there are existing libraries that perform these jobs in a better way.

### **Error Handling**

This is always a topic for discussion and could be great if you share your solutions here.
  
**<span style="color: #009933;">My strategy was to use callbacks,</span>** thus, if something happens in the data repository for example, the callback has 2 methods **<span style="color: #009933;">onResponse()</span>** and **<span style="color: #009933;">onError().</span>** The last one encapsulates exceptions in a wrapper class called **<span style="color: #009933;">&#8220;ErrorBundle&#8221;:</span>** This approach brings some difficulties because there is a chains of callbacks one after the other until the error goes to the presentation layer to be rendered. Code readability could be a bit compromised.
  
On the other side, I could have implemented an event bus system that throws events if something wrong happens but this kind of solution is like using a <a href="http://www.drdobbs.com/jvm/programming-with-reason-why-is-goto-bad/228200966" target="_blank">GOTO</a>, and, in my opinion, sometimes you can get lost when you&#8217;re subscribed to several events if you do not control that closely.

### **Testing**

Regarding testing, I opted for several solutions depending on the layer:

  * **<span style="color: #009933;">Presentation Layer:</span>** used android instrumentation and espresso for integration and functional testing.
  * **<span style="color: #009933;">Domain Layer:</span>** JUnit plus mockito for unit tests was used here.
  * **<span style="color: #009933;">Data Layer:</span>** Robolectric (since this layer has android dependencies) plus junit plus mockito for integration and unit tests.

### **Show me the code**

**<span style="color: #009933;">I know that you may be wondering where is the code, right?</span>** Well <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">here is the github link</a> where you will find what I have done. About the folder structure, something to mention, is that the different layers are represented using modules:

  * **<span style="color: #009933;">presentation:</span>** It is an android module that represents the presentation layer.
  * **<span style="color: #009933;">domain:</span>** A java module without android dependencies.
  * **<span style="color: #009933;">data:</span>** An android module from where all the data is retrieved.
  * **<span style="color: #009933;">data-test:</span>** Tests for the data layer. Due to some limitations when using Robolectric I had to use it in a separate java module.

### **Conclusion**

As Uncle Bob says, **<span style="color: #009933;">&#8220;Architecture is About Intent, not Frameworks&#8221;</span>** and I totally agree with this statement. Of course there are a lot of different ways of doing things (different implementations) and I&#8217;m pretty sure that you (like me) face a lot of challenges every day, but by using this technique, you make sure that your application will be:

  * **<span style="color: #009933;">Easy to maintain.</span>**
  * **<span style="color: #009933;">Easy to test.</span>**
  * **<span style="color: #009933;">Very cohesive.</span>**
  * **<span style="color: #009933;">Decoupled.</span>**

**<span style="color: #009933;">As a conclusion I strongly recommend you give it a try and see and share your results and experiences,</span>** as well as any other approach you’ve found that works better: we do know that **<span style="color: #009933;">continuous improvement</span>** is always a very good and positive thing.
  
I hope you have found this article useful and, as always, any feedback is very welcome.

### Source code

  1. <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Clean architecture github repository &#8211; master branch</a>
  2. <a href="https://github.com/android10/Android-CleanArchitecture/releases" target="_blank">Clean architecture github repository &#8211; releases</a>

### Further reading:

  1. <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android..the evolution</a>
  2. <a href="http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">Tasting Dagger 2 on Android</a>
  3. <a href="https://speakerdeck.com/android10/the-mayans-lost-guide-to-rxjava-on-android" target="_blank">The Mayans Lost Guide to RxJava on Android</a>
  4. <a href="https://speakerdeck.com/android10/it-is-about-philosophy-culture-of-a-good-programmer" target="_blank">It is about philosophy: Culture of a good programmer</a>

### **Links and Resources**

  1. <a href="http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html" target="_blank">The clean architecture by Uncle Bob</a>
  2. <a href="http://www.infoq.com/news/2013/07/architecture_intent_frameworks" target="_blank">Architecture is about Intent, not Frameworks</a>
  3. <a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter" target="_blank">Model View Presenter</a>
  4. <a href="http://martinfowler.com/eaaCatalog/repository.html" target="_blank">Repository Pattern by Martin Fowler</a>
  5. [Android Design Patterns Presentation](http://www.slideshare.net/PedroVicenteGmezSnch/)