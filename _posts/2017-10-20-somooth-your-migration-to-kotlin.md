---
id: 13
title: Smooth your migration to Kotlin
date: 2017-10-20T19:15:58+00:00
author: fernando
description: How to smoothly migrate your existing android java codebase to kotlin. These principles can also be applied to any existing technology being introduced.  
layout: post
permalink: /2017/10/20/smooth-your-migration-to-kotlin/
image: assets/images/smooth_kotlin_featured.jpg
comments: false
featured: false
hidden: false
categories: [ android, mobile, java, kotlin, functional, programming ]
tags:
  - android
  - androiddev
  - process
  - agile
  - migration
  - transition
  - developer
  - developers
  - development
  - java
  - kotlin
  - mobile
  - mobile dev
  - programmer
  - programming
  - reactive programming
  - rxjava
  - testing
  - continuous learning
  - new technologies
---
First of all, **I would like to give credit to who deserves it,** and in this occasion wanna say thanks to <a href="https://twitter.com/Mauin" target="_blank">@Mauin</a> for taking the leadership on the process and let me join the party. Was a real pleasure. Also, I would like to mention, that in this article there is NO source code involved and here is the reason: **It is not about technical implementation, it is about the process, philosophy and all the moving parts involved.**

Same principles apply not only when introducing a new programming language, but also any new technology.
 
Actually the motivation behind this writing <a href="https://twitter.com/fernando_cejas/status/920591431730900992" target="_blank">came from a tweet</a> and a couple of discussions and constructive feedback around it:

![fernando-cejas](/assets/images/smooth_kotlin_01.png)

So in this article I will bring up insights (and opinions) on how to introduce <a href="https://kotlinlang.org/" target="_blank">Kotlin</a> into your existing Android Java codebase. **All this material comes from experiences and real facts,** which from my perspective, is the best way to share knowledge and lessons learned. So let's get started.


## The movitation

Let's say we heard about this new thing called <a href="https://kotlinlang.org/" target="_blank">Kotlin</a> (mainly) being used for Android development nowadays. Many people are talking about it, so we start to dive a little bit deeper and see potential in it, so in the end we decide to bring the topic to the table in our next team meeting.

There should be always a motivation with strong arguments for betting on new technologies, and **those reasons do NOT include**:

  * Because everyone is using it.
  * Because it is trendy.
  * Because it is cool.

![fernando-cejas](/assets/images/smooth_kotlin_02.jpg)

**Keep in mind that we are taking risks at a technical and business level.** So if we do not do our homework first, we might run into the unknown with disastrous consequences, which are not easy to rollback, consume development time and, for instance, money for the organization.

To avoid these issues the first step should be to consider a bunch of basic questions:

  * **Is the technology mature enough?**
  * **Has the technology support from the community?**
  * **Is out there other companies which have already adopted it?**
  * **Do we have experts in the team who can lead/guide us in the process?**
  * **Is the team interested in learning this new programming language?**

These are, in my opinion, the first answers we need, in order to move forward to the next stage.


## Involving the team

The advice here is to involve our team in the process as much as we can. It is important that everyone feels part of it by providing **feedback, advice and opinions.** We want to make sure we are in the same boat on our path.

There are a a couple of things we can do to collect such information and participation:

  * Collective meetings for discussions.
  * Hacker time for spiking.
  * Pair programming on a pet project as example.
  * RFCs through a shared document.
  * Some basic workshop or presentation after investigation.

**We want everyone to commit as much as possible** which will help the decision making and the collection of information about potential issues we might run into.

Something we did was to also talk to the iOS Team, since they had gone through a similar process when migrating from Objective C to Swift: super valuable feedback and experience.

![fernando-cejas](/assets/images/smooth_kotlin_03.png)


## Resisting the inevitable

In the past (when Kotlin was not officially supported by Google) we had a few failed attempts which caused uncertainty and frustration in the team due to these doubts:

  * **Does it integrate well with our current Continuous Integration Pipeline?**
  * **What about tools availability? Like static analysis for example.**
  * **It is still in Beta and APIs are likely to change without backward compatibility in mind.**

But... I think it is time to invoke Pablo Neruda here which reflects the impact Kotlin is creating in the development community:

![fernando-cejas](/assets/images/smooth_kotlin_04.png)

With that being said, nowadays there are not many reasons to NOT introduce Kotlin, or at least the upsides outweigh the downsides. Anyway, that does not mean we do NOT have to focus on the pros and cons of introducing a new technology based on:

  * **Our priorities as a Team/Company:** we need time and we might slow down feature development in the beginning, although that is going to pay back in the future.
  * **The state of our current codebase:** in our first attempt we were deep in changing our architecture and killing legacy code. Introducing a new programming language would increase our Lava Anti-Pattern.

At <a href="https://soundcloud.com/" target="_blank">SoundCloud</a>, after dealing with these 2 aspects we decided it was worth to move forward, so we came up with a bunch of bullet points (a couple of examples below) we would need to address in the next stage:

  * Make the Java-Kotlin interop as easy as possible.
  * When converting `@AutoValue` classes to `data class`, add static factory methods back in the same way. Ex. `@JvmStatic` functions in `companion objects` of the `data class`: Thus, we use the same methods in Java to create those objects. From Kotlin we can directly call the constructor.
  * If `data class` contains nullable types and those are still used from Java, we should add getters that directly return `Optional<T>` instead of the `Nullable` type, which we might miss in the Java context.
  * Keep in mind: Kotlins types don't always map directly to the Java types. Kotin `Long` is `long` in Java, which doesn't have all the compare methods.

**This way, we prepare the terrain by detecting potential issues and solutions to the most common problems in our codebase.**


## Our First PR

At this point, we detected possible problems to solve and it was time to officially get our hands dirty. This is an important stage because we are translating our previous investigation into code.

**In my opinion this should be done with Pair Programming.** There should be discussions, suggestions and pair reviews from the rest of the team. It is a good idea to put a deadline until when the PR will be open, otherwise discussions become endless.

If you are wondering what to work on for your first Kotlin PR, I would say that just **pick a simple functionality** (a screen) in your app and **refactor it end to end.** This way, you start your job vertically, and gradually move to the rest of features. **Divide and conquer, remember?**

Along the way, we did not only write code, but also performed tests, for example compilation time. The more information we collect sooner, the better.

![fernando-cejas](/assets/images/smooth_kotlin_05.png)

Finally it was time to push the button and commit our first Kotlin PR. It was a real pleasure to presence that moment (again thanks <a href="https://twitter.com/Mauin" target="_blank">@Mauin</a>!). **Now we are happy kotlin coders!**

![fernando-cejas](/assets/images/smooth_kotlin_06.png)


## More power implies more responsibility

Everything does not end up once you merge your first piece of Kotlin code. **We are embracing a new language (a new technology)** so there should be commitment after it. The main idea is to establish rules we must follow in order to fully transition from the old stack to the new one.

Here are some a few examples to make this process smooth without impacting productivity too much:

  * **Each new piece of code introduced to our codebase should be written in the new language.**
  * **Each piece of code touched should be migrated to the new language, unless it is a critical bug and needs rapid reaction.**
  * **When reviewing PRs, we should enforce the team to follow these rules.**
  * **If newly converted Kotlin code is still called from Java, try to keep the interface changes for the Java callers to a minimum.**

My suggestion is to also allocate time (when possible) every sprint to migrate and refactor features. Maybe it is also a good idea to address technical debt too and write it in Kotlin: kill two birds with only one stone (legacy code and Java).


## Continuous Learning

This an exciting process and we should keep it up by **constantly learning,** improving and addressing together problems we find along the way. **Sharing and transfer of knowledge is key.** As mentioned earlier, what worked for us, were discussions in our collective weekly meeting, presentations, pair programming and little workshops. 

![fernando-cejas](/assets/images/smooth_kotlin_07.png)

As a colleague of mine say, do not put things in cold water and always encourage continuous improvement.


## Benefits and key points

Here I wanted to include facts and situations we are getting out of this transition:

  * Boost to developer morale, karma and motivation.
  * Increased null safety in the Android codebase (due to nullable/non-nullable types).
  * Less boilerplate that has to be written, making Pull-Requests smaller and easier to review and the codebase as a whole easier to understand.
  * In the long term, being able to remove some third-party dependencies (e.g. Retrolambda, ButterKnife, AutoValue).
  * In the middle/long term, it also keeps us as a competitive and modern organization betting on the latest/new technologies, in order to attract future talent to the Company/Team.
  * Staying up-to-date with the Android developer community.
  * Testing in Kotlin might help, it is better than nothing but you will not appreciate the real power until you solve real world problems with it.


## Conclusion

As we have seen, it is never straightforward to introduce such big changes. **Also, the bigger the company is, the more difficult, due to its moving parts, team size and amount of code generated every day.**

That is why it is important to come up with a **plan** to make the process as smooth as possible.

This is all I have to offer for now, <a href="https://twitter.com/fernando_cejas" target="_blank">ping me on Twitter</a> for discussions/feedback. **Happy kotlin coding!**