---
id: 4
title: 'Architecting Android...The clean way?'
date: 2014-09-03T01:45:54+00:00
author: fernando
description: The purpose of this article is to show you a little approach based on Clean Architecture that I had in mind in the last few months plus all the stuff I have learnt from investigating and implementing it.
layout: post
permalink: /2014/09/03/architecting-android-the-clean-way/
image: assets/images/clean_architecture_clean_way_featured.jpg
comments: false
featured: false
hidden: false
categories: [ android, mobile, java, architecture, oop, programming ]
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
Over the last months and after having a few android discussions at <a href="http://corporate.tuenti.com/en/dev/blog" target="_blank">Tuenti</a> with colleagues like <a href="https://twitter.com/pedro_g_s" target="_blank">@pedro_g_s</a> and <a href="https://twitter.com/flipper83" target="_blank">@flipper83</a>, I have decided that was a good time to write an article about **architecting android applications.**
  
The purpose of it is to show you a **little approach I had in mind** in the last few months plus all the stuff I have learnt from investigating and implementing it.


## Getting Started

**We know that writing quality software is hard and complex:** It is not only about satisfying requirements, also should be **robust**, **maintainable**, **testable** and **flexible** enough to adapt to growth and change. This is where "**the clean architecture**" comes up and could be a good approach for using when developing any software application.
  
The idea is simple: clean architecture stands for a group of practices that produce systems that are:

  * **Independent of Frameworks.**
  * **Testable.**
  * **Independent of UI.**
  * **Independent of Database.**
  * **Independent of any external agency.**

![clean_architecture_android](/assets/images/clean_architecture_clean_way_01.png)

It is not a must to use only 4 circles (as you can see in the picture), because they are only schematic but you should take into consideration the **Dependency Rule:** source code dependencies can only point inwards and nothing in an inner circle can know anything at all about something in an outer circle.

Here is some vocabulary that is relevant for getting familiar and understanding this approach in a better way:

  * **Entities:** These are the business objects of the application.
  * **Use Cases:** These use cases orchestrate the flow of data to and from the entities. Are also called Interactors.
  * **Interface Adapters:** This set of adapters convert data from the format most convenient for the use cases and entities. Presenters and Controllers belong here.
  * **Frameworks and Drivers:** This is where all the details go: UI, tools, frameworks, etc.

For a better and more extensive explanation, refer to <a href="http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html" target="_blank">this article</a> or <a href="http://vimeo.com/43612849" target="_blank">this video</a>.


## Our Scenario

**I will start with a simple scenario to get things going:** simply create an small app that shows a list of friends or users retrieved from the cloud and, when clicking any of them, a new screen will be opened and for instance, show more details for that user. 

Here is a quick video:

<center><iframe width="640" height="480" src="//www.youtube.com/embed/XSjV4sG3ni0" frameborder="0" allowfullscreen="allowfullscreen"></iframe></center>


## Android Architecture

**The purpose is the separation of concerns by keeping the business rules not knowing anything at all about the outside world,** thus, they can can be tested without any dependency to any external element.
  
To achieve this, my proposal is about breaking up the project into **3 different layers**, in which each one has its own purpose and works separately from the others.
  
It is worth mentioning that **each layer uses its own data model** so this independence can be reached  (you will see in code that a data mapper is needed in order to accomplish data transformation, a price to be paid if you do not want to cross the use of your models over the entire application).
  
Here is an schema so you can see how it looks like:

![clean_architecture_android](/assets/images/clean_architecture_clean_way_02.png)

**NOTE:** I did not use any external library (except gson for parsing json data and junit, mockito, robolectric and espresso for testing). The reason is that it makes the example clearer. Anyway do not hesitate to add ORMs for storing disk data or any dependency injection framework or whatever tool or library you are familiar with, that could make your life easier. 

**REMEMBER:** reinventing the wheel is not a good practice.


## Presentation Layer

Is here, where the logic related with views and animations happens. It uses no more than a Model View Presenter (<a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter" target="_blank">MVP</a> from now on), but you can use any other pattern like MVC or MVVM. I will not get into details on it, but here **fragments and activities are only views**, there is no logic inside them other than UI logic, and this is where all the rendering stuff takes place.
  
**Presenters in this layer are composed with interactors (use cases) that perform the job in a new thread outside the main android UI thread**, and come back using a callback with the data that will be rendered in the view.

![clean_architecture_android](/assets/images/clean_architecture_clean_way_03.png)

If you want a cool example about <a href="https://github.com/pedrovgs/EffectiveAndroidUI/" target="_blank">Effective Android UI</a> that uses **MVP** and **MVVM**, take a look at what my friend Pedro Gómez has done.


## Domain Layer

**Business rules here:** all the logic happens in this layer. Regarding the android project, you will see all the interactors (use cases) implementations here as well.
  
**This layer is a pure java module** without any android dependencies. All the external components use interfaces when connecting to the business objects.

![clean_architecture_android](/assets/images/clean_architecture_clean_way_04.png)


## Data Layer

All data needed for the application comes from this layer through a **UserRepository** implementation (the interface is in the domain layer) that uses a <a href="http://martinfowler.com/eaaCatalog/repository.html" target="_blank">Repository Pattern</a> with a strategy that, through a factory, picks different data sources depending on certain conditions.
  
For instance, when getting a user by id, the disk cache data source will be selected if the user already exists in cache, otherwise the cloud will be queried to retrieve the data and later save it to the disk cache.
  
The idea behind all this is that **the origin of the data is transparent to the client**, which does not care if the data is coming from memory, disk or the cloud, the only truth is that the information will arrive and will be gotten.

![clean_architecture_android](/assets/images/clean_architecture_clean_way_05.png)


**NOTE:** In terms of code I have implemented a very simple and primitive disk cache using the file system and android preferences, it was for learning purpose. Remember again that you **SHOULD NOT REINVENT THE WHEEL** if there are existing libraries that perform these jobs in a better way.


## Error Handling

This is always a topic for discussion and could be great if you share your solutions here.
  
My strategy was to use callbacks, thus, if something happens in the data repository for example, the callback has 2 methods ```onResponse()``` and ```onError()```. 

The last one encapsulates exceptions in a wrapper class called "**ErrorBundle**": This approach brings some difficulties because there is a chains of callbacks one after the other until the error goes to the presentation layer to be rendered. **Code readability could be a bit compromised.**
  
On the other side, I could have implemented an event bus system that throws events if something wrong happens, but this kind of solution is like using a <a href="http://www.drdobbs.com/jvm/programming-with-reason-why-is-goto-bad/228200966" target="_blank">GOTO</a>, and, in my opinion, sometimes you can get lost when you're subscribed to several events if you do not control them closely.


## Testing

Regarding testing, I opted for several solutions depending on the layer:

  * **Presentation Layer:** used android instrumentation and espresso for integration and functional testing.
  * **Domain Layer:** JUnit plus mockito for unit tests was used here.
  * **Data Layer:** Robolectric (since this layer has android dependencies) plus junit plus mockito for integration and unit tests.


## Show me the code

I know that you may be wondering where is the code, right? Well <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">here is the github link</a> where you will find what I have done. 

About the folder structure, something to mention, is that the different layers are represented using modules:

  * **presentation:** It is an android module that represents the presentation layer.
  * **domain:** A java module without android dependencies.
  * **data:** An android module from where all the data is retrieved.
  * **data-test:** Tests for the data layer. Due to some limitations when using Robolectric I had to use it in a separate java module.


## Conclusion

As Uncle Bob says, "**Architecture is About Intent, not Frameworks**" and I totally agree with this statement. Of course there are a lot of different ways of doing things (different implementations) and I'm pretty sure that you (like me) face a lot of challenges every day, but by using this technique, you make sure that your application will be:

  * **Easy to maintain.**
  * **Easy to test.**
  * **Very cohesive.**
  * **Decoupled.**

As a conclusion I strongly recommend you give it a try and see and share your results and experiences, as well as any other approach you’ve found that works better: **we do know that continuous improvement is always a very good and positive thing.**
  
I hope you have found this article useful and, as always, any feedback is more than welcome.


## Source code

  * <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">Clean architecture github repository &#8211; master branch</a>
  * <a href="https://github.com/android10/Android-CleanArchitecture/releases" target="_blank">Clean architecture github repository &#8211; releases</a>


## Further reading:

  * <a href="https://fernandocejas.com/2018/05/07/architecting-android-reloaded/" target="_blank">Architecting Android..reloaded</a> 
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