---
id: 15
title: 'Architecting Android...Reloaded'
date: 2018-05-07T14:06:11+00:00
author: Fernando Cejas
description: Clean architecture evolution on Android. Hints for Functional Programming, OOP, Error Handling, Modularization and Patterns. Written in Kotlin.   
layout: post
permalink: /2018/05/07/architecting-android-reloaded/
categories:
  - Android
  - Development
  - Java
  - Kotlin
  - Software Architecture
  - Functional Programming
  - Error Handling
  - Modularization
tags:
  - android
  - androiddev
  - architecture
  - clean architecture
  - dagger
  - dependency injection
  - development
  - kotlin
  - java
  - mobile
  - mobile dev
  - programmer
  - programming
  - reactive
  - reactive programming
  - coroutines
  - fp
  - oop
  - functional programming
  - continuous learning
  - agile
  - mobile
  - error handling
  - Modularization
---
<p class="justify"><span class="underlinetext">After a long time</span> I decided to write again about <span class="boldtext">Architecture on Android Applications.</span> The reason? Mainly <span class="boldtext">feedaback</span> from the community and <span class="boldtext">lessons learned.</span> But even though a lot has been said since the early days when <span class="boldtext">Clean Architecture</span> became popular in <span class="boldtext">Mobile Development,</span> there is always room for <span class="boldtext">improvement</span> and <span class="boldtext">evolution.</span></p>
 
<p class="justify">In order to get started and get things easier, I will assume <span class="underlinetext">you have already read</span> these old but still valid <span class="underlinetext">blog posts:</span></p>

  * <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android...The clean way?</a>
  * <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android...The evolution.</a>

<p class="justify">Based on the above articles clean architecture example, <span class="boldtext">there is a clear evolution in the codebase,</span> especially because nowadays with applications being key at a business level, more than ever, there is a need to scale, modularize and organize teams around <span class="boldtext">Mobile Development (mainly due to its complexity).</span></p>

<p class="justify">Thus, the idea is to come up with an (<span class="underlinetext">elegant?</span>) solution which will make our life easier in terms of:</p>

  * <span class="boldtext">Problem solving.</span>
  * <span class="boldtext">Scalability.</span>
  * <span class="boldtext">Modularization.</span>
  * <span class="boldtext">Testability.</span>
  * <span class="boldtext">Independence of frameworks, UI and Databases.</span>

<br>
![fernando-cejas](/assets/images/clean_architecture_reloaded_main.png){:class="img-responsive"}
<br>

<p class="justify">This is the big picture, which should look familiar if you have been using <span class="boldtext">Clean Architecture in your Android Applications.</span></p>

<br>
# Our scenario

<p class="justify">A <span class="underlinetext">simple</span> <span class="boldtext">Movies Android Application</span> (any similarities with reality is a mere coincidence).</p>

<p class="justify"><span class="underlinetext">Written in Kotlin:</span> not much to say other than that we want to leverage a <span class="boldtext">modern language's features like inmutability, conciseness, functional programming, etc.</span></p>

<p class="justify">With the <span class="underlinetext">following flow:</span></p>

![fernando-cejas](/assets/images/clean_architecture_reloaded_screenshots.png){:class="img-responsive"}

<p class="justify">Where we have <span class="underlinetext">3 main use cases:</span></p>

  * <span class="boldtext">Get a list of movies.</span>
  * <span class="boldtext">Show details for an specific clicked movie.</span>
  * <span class="boldtext">Play a movie.</span>

<p class="justify">As usual, the <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin" target="_blank">source code is available on github.</a></p> 

<br>
# General Architecture

<p class="justify">The general principle is to use a <span class="boldtext">basic 3 tiers architecture.</span> The good thing about it, is that it is very <span class="boldtext">easy to understand</span> and many people are familir with it. <span class="boldtext">So we will break down our solution into layers</span> in order to respect the <span class="boldtext">dependency rule</span> (<span class="underlinetext">where dependencies flow in one direction:</span> check above the rounded clean architecture graph):</p>

<br>
![fernando-cejas](/assets/images/clean_architecture_reloaded_layers.png){:class="img-responsive"}
<br>

<p class="justify"><span class="boldtext">Nothing new here</span> if we keep in mind my <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">previous posts,</a> but <span class="underlinetext">do not stop reading yet.</span> Let's dive deeper and go piece by piece for a better understanding.</p>

<br>
# Domain Layer: Functional Use Cases

<p class="justify"><span class="boldtext">A use case is an intention,</span> in other words, something we want to do in our application, one of our main players. <span class="boldtext">And its main responsibility is to orchestrate our domain logic</span> and its connection with both <span class="boldtext">UI</span> and <span class="boldtext">Data</span> layers.</p> 

<p class="justify">By using <span class="boldtext">the power of Kotlin</span> and its treatment of <span class="boldtext">functions as first class citizens</span> (more coming up shortly), we have in our framework a <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/core/interactor/UseCase.kt" target="_blank">UseCase</a> abstraction, which acts as a <span class="boldtext">contract</span> for all the use cases in our application.</p>

```java
abstract class UseCase<out Type, in Params> where Type : Any {

    abstract suspend fun run(params: Params): Either<Failure, Type>

    fun execute(onResult: (Either<Failure, Type>) -> Unit, params: Params) {
        val job = async(CommonPool) { run(params) }
        launch(UI) { onResult.invoke(job.await()) }
    }
}
```

### What is going on here? 

<p class="justify">We have an <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/core/interactor/UseCase.kt" target="_blank">abstract class</a> which takes <span class="boldtext">2 generic parameters:</span></p> 
  
  * ```<out Type>```: a return type which is <span class="boldtext">the result of the executed use case.</span>
  * ```<in Params>```: a parameters class which will be consumed inside the <span class="boldtext">run()</span> function in case we need <span class="boldtext">extra data for our use case.</span>

<p class="justify">The <span class="boldtext">execute()</span> function is where all the magic happens:</p>

  * <p class="justify">We pass a <span class="boldtext">"onResult"</span> function as a parameter which takes an <span class="boldtext">Either<Failure, Type></span> and returns <span class="boldtext">Unit</span> (in the <span class="underlinetext">Error Handling</span> section I will extend the explanation for <span class="boldtext">Either<L, R></span>, <span class="underlinetext">so be patient please</span> :)). The good thing is that the caller of the <span class="boldtext">UseCase</span> is actually establishing the desired behaviour by passing this <span class="boldtext">inmutable function (onResult),</span> thus, <span class="underlinetext">avoiding any internal exposure or side effects</span> if we were passing objects <span class="boldtext">(one of the benefits of FP, more coming up).</span></p> 
  * <p class="justify">Also, <span class="boldtext">by using Kotlin coroutines we invoke the passed "onResult" function in a different thread,</span> so from this point on, we are safe to write our code in a <span class="boldtext">synchronous</span> fashion. <span class="boldtext">The result will be posted on the Android Main UI Thread.</span></p>

<p class="justify"><span class="boldtext">abstract suspend fun run(params: Params)</span> is what we have to override when extending the <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/core/interactor/UseCase.kt" target="_blank">UseCase<out Type, in Params></a> abstraction, so for instance, this is how our <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/features/movies/GetMovies.kt" target="_blank">GetMovies</a> use case looks like:</p>

```java
class GetMovies
@Inject constructor(private val moviesRepository: MoviesRepository) : UseCase<List<Movie>, None>() {

    override suspend fun run(params: None) = moviesRepository.movies()
}
```

<p class="justify">In this example we are delegating movies retrieval to a <span class="boldtext">Repository.</span> <span class="underlinetext">Easy right?</span></p>

<br>
# UI Layer: From MVP to MVVM

<p class="justify">The <span class="boldtext">Model-View-ViewModel Pattern (MVVM) provides a clean separation of concerns between user interface and domain logic.</span></p>

<p class="justify">It has 3 main components: <span class="boldtext">the model, the view, and the view model.</span> There are <span class="underlinetext">relationships between them,</span> although <span class="boldtext">each serves a distinct and separate role:</span></p>

<br>
![fernando-cejas](/assets/images/clean_architecture_reloaded_mvvm.png){:class="img-responsive"}
<br>

<p class="justify">At the highest level, <span class="boldtext">the view "knows about" the view model, and the view model "knows about" the model, but the model is unaware of the view model, and the view model is unaware of the view.</span> The view model isolates the view from the model classes and allows the model to evolve independently of the view.</p>

<p class="justify"><span class="underlinetext">At implementation level,</span> in our example, <span class="boldtext">MVVM</span> is accomplished by the usage of <a href="https://developer.android.com/topic/libraries/architecture/" target="_blank">Architecture Components</a>, which its main advantage is to <span class="boldtext">handle configuration changes when the screen rotates,</span> something that has given us many <span class="boldtext">headaches as android developers</span> (I guess you know what I'm talking about).</p>

<p class="justify"><span class="boldtext"><span class="underlinetext">Disclaimer:</span></span> that does not mean we have to no longer care about lifecycles, but <span class="underlinetext">it is way easier.</span></p>

<p class="justify">A comment on <span class="boldtext">MVP (Model View Presenter)</span> from <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">previous example:</a> <span class="boldtext">I found tricky to avoid leaks due to activities and fragments being recreated</span> so I used a <span class="underlinetext">poor man solution:</span> retain fragments.</p> 

<p class="justify">However, I ran into these sort of situations anyway. <span class="boldtext">This is the reason why I decided to go and give a try to MVVM.</span></p> 

### Let's see what changed with MVVP from previous sample and how it works:

<br>
![fernando-cejas](/assets/images/clean_architecture_reloaded_mvvm_app.png){:class="img-responsive"}
<br>

* <span class="boldtext">Fragments act as views,</span> where all the logic related to displaying data on the screen happens. 
* <span class="boldtext">Fragments also know about ViewModels,</span> they actually subscribe to ViewModels.
* <span class="boldtext">ViewModels</span> contain <span class="boldtext">LiveData</span> objects and references to <span class="boldtext">UseCases.</span>
* <span class="boldtext">UseCases</span> update <span class="boldtext">LiveData</span> which react to those changes and send notifications to <span class="boldtext">ViewModels.</span>
* <span class="boldtext">ViewModels</span> talk to subscribed <span class="boldtext">Fragments</span> in order to update the <span class="boldtext">UI.</span> 

<p class="justify">In order to see all these pieces working together, <span class="underlinetext">let's see some code.</span></p>

<span class="boldtext">ViewModel containing LiveData</span> and updating by calling <span class="boldtext">UseCase.execute()</span> function:

```java
class MoviesViewModel
@Inject constructor(private val getMovies: GetMovies) : BaseViewModel() {

    var movies: MutableLiveData<List<MovieView>> = MutableLiveData()

    fun loadMovies() = getMovies.execute({ it.either(::handleFailure, ::handleMovieList) }, None())

    private fun handleMovieList(movies: List<Movie>) {
        this.movies.value = movies.map { MovieView(it.id, it.poster) }
    }
}
```

<p class="justify"><span class="boldtext">Fragment subscribing to the above ViewModel on onCreate().</span> I used a <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/core/extension/Fragment.kt#L33" target="_blank">couple of tricks</a> with <span class="boldtext">extension functions</span> to get rid of some <span class="boldtext">verbosity</span> and <span class="boldtext">boilerplate.</span></p>

```java
class MoviesFragment : BaseFragment() {

    @Inject lateinit var navigator: Navigator
    @Inject lateinit var moviesAdapter: MoviesAdapter

    private lateinit var moviesViewModel: MoviesViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        appComponent.inject(this)

        //subscribtion to LiveData in MoviesViewModel
        moviesViewModel = viewModel(viewModelFactory) {
            observe(movies, ::renderMoviesList)
            failure(failure, ::handleFailure)
        }
    }
	...
}
```

<br>
# Data Layer: Repository Pattern to the rescue

<p class="justify"><span class="boldtext">Anything new here</span> in comparison with the <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">previous example</a> since I have had <span class="boldtext">really good results</span> with it.</p>

<p class="justify"><span class="boldtext"><span class="underlinetext">To keep in mind:</span> At its core the Repository pattern is a simple interface.</span> It exists as a layer between our <span class="boldtext">domain</span> and our <span class="boldtext">data</span> so that our <span class="boldtext">logic doesn't need to be concerned with the implementation</span> of different data sources: <span class="underlinetext">Network, Database or Memory.</span></p> 

<br>
![fernando-cejas](/assets/images/clean_archictecture_reloaded_repository.png){:class="img-responsive"}
<br>

<p class="justify">In the following <span class="underlinetext">chunk of code</span> we can see our <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/features/movies/MoviesRepository.kt" target="_blank">MoviesRepositoy</a> contract:</p>

```java
interface MoviesRepository {
    fun movies(): Either<Failure, List<Movie>>
    fun movieDetails(movieId: Int): Either<Failure, MovieDetails>
}
```

<p class="justify"><a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/features/movies/GetMovies.kt#L23" target="_blank">In our example</a> we usually inject a <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/features/movies/MoviesRepository.kt" target="_blank">Repositoy</a> as a collaborator for our <span class="boldtext">UseCases implementations.</span></p>

<br>
# Functional Error Handling

<p class="justify"><span class="boldtext">Overall Error/Exception handling should be taken care at design and not at implementention level,</span> and in my opinion, one of the biggest mistakes we make as developers (lesson learned). That is why it is <span class="boldtext">important to have a framework in place for this purpose.</span></p>

### What happens with traditional Error Handling? 

<p class="justify"><span class="boldtext">Observing exceptions (try/catch blocks) and making decisions based on it that changes the control flow is a bad practice: it creates unprediction, affects our resilience and debugging becomes difficult, especially in concurrent environments.</span> Plus going back to C-style error handling, using error codes which need to be checked by convention could be a nightmare.</p>

<p class="justify">With that being said, we have seen we are using <span class="boldtext">Either<L, R></span> as a return type in our <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/core/interactor/UseCase.kt" target="_blank">UseCase</a> abstraction:</p>

```java
abstract suspend fun run(params: Params): Either<Failure, Type>
```

### So let me introduce Either 

<p class="justify"><span class="boldtext">Either<L, R></span> is referred as a <span class="boldtext">disjoint function,</span> which means that this structure is designed to hold <span class="boldtext">either a Left&lt;T&gt; or Right&lt;T&gt; value but never both.</span> It is a funcional programming <span class="boldtext">monadic type</span> not yet existent in the Kotlin Standard Library.</p>

<p class="justify"><span class="boldtext">Here is a simple implementation</span> which perfectly fulfills my needs and it is <span class="boldtext">easy to understand and use</span> (ideas taken from <a href="https://proandroiddev.com/kotlins-nothing-type-946de7d464fb" target="_blank">Alex Hart</a>):</p>

```java
/**
 * Represents a value of one of two possible types (a disjoint union).
 * Instances of [Either] are either an instance of [Left] or [Right].
 * FP Convention dictates that [Left] is used for "failure" and [Right] is used for "success".
 *
 * @see Left
 * @see Right
 */
sealed class Either<out L, out R> {
    /** * Represents the left side of [Either] class which by convention is a "Failure". */
    data class Left<out L>(val a: L) : Either<L, Nothing>()
    /** * Represents the right side of [Either] class which by convention is a "Success". */
    data class Right<out R>(val b: R) : Either<Nothing, R>()

    val isRight get() = this is Right<R>

    val isLeft get() = this is Left<L>

    fun either(fnL: (L) -> Any, fnR: (R) -> Any): Any =
            when (this) {
                is Either.Left -> fnL(a)
                is Either.Right -> fnR(b)
            }

    fun <T> flatMap(fn: (R) -> Either<L, T>): Either<L, T> {...}
    fun <T> map(fn: (R) -> (T)): Either<L, T> {...}
}
```

<p class="justify">Let me also quote <a href="https://danielwestheide.com/" target="_blank">Daniel Westheide</a> (a Scala guru and good person, whom I bumped into at SoundCloud) in one of <span class="underlinetext">his amazing blog posts:</span></p>

<p class="justify"><span class="boldtext">"</span>There is nothing in the semantics of the <span class="boldtext">Either<L, R></span> type that specifies one or the other sub type to represent <span class="boldtext">an error or a success,</span> respectively. In fact, <span class="boldtext">Either is a general-purpose type</span> for use whenever you need to deal with situations where the result can be of one of two possible types.</p>

<p class="justify"><span class="boldtext">Nevertheless, error handling is a popular use case for it, and by convention, when using it that way, the Left&lt;T&gt; represents the error case, whereas the Right&lt;T&gt; contains the success value."</span></p>

<p class="justify">And please do not forget to read <span class="underlinetext">his entire serie of scala posts</span> to expand your horizons even more (getting ideas from other languages is always a +1):</p>

  * <a href="http://danielwestheide.com/blog/2012/12/26/the-neophytes-guide-to-scala-part-6-error-handling-with-try.html" target="_blank">Error Handling With Try.</a>
  * <a href="http://danielwestheide.com/blog/2013/01/02/the-neophytes-guide-to-scala-part-7-the-either-type.html" target="_blank">The Either Type.</a>

### What about our code sample then? 

In the <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/features/movies/GetMovies.kt" target="_blank">GetMovies</a> use case, <span class="underlinetext">at implementation level,</span> we always return an <span class="boldtext">Either&lt;Failure, List&lt;Movie&gt;&gt;</span> up starting from the data layer, all the way up to our <span class="boldtext">MoviesViewModel</span> which updates <span class="boldtext">"either"</span> the failure <span class="boldtext">LiveData&lt;Failure&gt;</span> (in case of <span class="underlinetext">failure,</span> <span class="boldtext">Left&lt;T&gt;</span>) or the movies <span class="boldtext">LiveData&lt;List&lt;MovieView&gt;&gt;</span>(<span class="underlinetext">success,</span> <span class="boldtext">Right&lt;T&gt;</span>):


```java
class MoviesViewModel
@Inject constructor(private val getMovies: GetMovies) {

    var movies: MutableLiveData<List<MovieView>> = MutableLiveData()
    var failure: MutableLiveData<Failure> = MutableLiveData()

    fun loadMovies() = getMovies.execute({ it.either(::handleFailure, ::handleMovieList) }, None())

    private fun handleMovieList(movies: List<Movie>) {
        this.movies.value = movies.map { MovieView(it.id, it.poster) }
    }

    private fun handleFailure(failure: Failure) {
        this.failure.value = failure
    }
}
```
 
<p class="justify"><span class="underlinetext">At a View level</span> (our <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/features/movies/MoviesFragment.kt" target="_blank">MoviesFragment</a>), <span class="boldtext">we subscribe to the updates coming from the view model:</span></p>

```java
moviesViewModel = viewModel(viewModelFactory) {
    observe(movies, ::renderMoviesList)
    failure(failure, ::handleFailure)
} 
```

<p class="justify">And this is how <span class="underlinetext">handleFailure()</span> looks like for <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/core/exception/Failure.kt" target="_blank">Failure</a> treatment:</p>

```java
private fun handleFailure(failure: Failure?) {
    when (failure) {
        is NetworkConnection -> renderFailure(R.string.failure_network_connection)
        is ServerError -> renderFailure(R.string.failure_server_error)
        is ListNotAvailable -> renderFailure(R.string.failure_movies_list_unavailable)
    }
}
```

<p class="justify">By the way, <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/main/kotlin/com/fernandocejas/sample/core/exception/Failure.kt" target="_blank">Failure</a> is a sealed class which <span class="boldtext">offers global default Failures:</span></p>

```java
/**
 * Base Class for handling errors/failures/exceptions.
 * Every feature specific failure should extend [FeatureFailure] class.
 */
sealed class Failure {
    class NetworkConnection: Failure()
    class ServerError: Failure()

    /** * Extend this class for feature specific failures.*/
    abstract class FeatureFailure: Failure()
}
```

<p class="justify">I hope now the usage of <span class="boldtext">Either<L, R></span> is clearer and you understand <span class="boldtext">the reason and benefits of this applied technique.</span></p>

<br>
# First steps to Modularization

<p class="justify">First, in my defense, this post is <span class="boldtext">NOT</span> about this specific topic (which by the way, is huge) but I want to write down some <span class="boldtext">hints based in past experiences,</span> in order to get started. From my perspective, sooner or later, this is the way to go, and <span class="boldtext">a good architecture should help out</span> to achieve this goal.</p>

### What is Modularization? 

<p class="justify"><span class="boldtext">Modularization is the process of separating and creating clear boundaries between logical components of code.</span></p>

<p class="justify"><a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">If you did your homework and checked my previous posts about clean architecture</a>, you might have seen that I used android modules for <span class="underlinetext">representing each layer involved in the architecture.</span></p>

<p class="justify">A recurring question in discussions was: <span class="boldtext">Why?</span> The answer is simple... <span class="boldtext">Wrong technical decision:</span> I relied on different modules in order to be more strict with <span class="boldtext">the dependency rule</span> by establishing borders and, thus, make it more difficult to break it.</p>

<p class="justify"><span class="boldtext">But power comes with big responsibilities,</span> and even though this worked pretty well in the beginning, the sample was still a <span class="boldtext">MONOLITH</span> and it would bring problems when scaling up:</p>

  * <span class="boldtext">when modifying or adding a new functionality:</span> we had to touch every single module/layer (strong dependencies/coupling between modules).
  * <span class="boldtext">conflicts when developers working in the codebase</span> (the bigger the team, the more conflicts, especially with PRs and git).

### Embrace Application Modularization

<p class="justify"><span class="underlinetext">My first tip</span> to favor modularization is to <span class="boldtext">organize packages by features,</span> this way we accomplish:</p>

  * <span class="boldtext">Higher Modularity.</span>
  * <span class="boldtext">High Cohesion.</span>
  * <span class="boldtext">Easier Code Navigation.</span>
  * <span class="boldtext">Minimizes Scope.</span>
  * <span class="boldtext">Isolation and Encapsulation.</span>

<p class="justify"><span class="boldtext">Code/package organization is one of the key factors of a good architecture:</span> package structure is the very first thing encountered by a programmer when browsing source code. <span class="boldtext">Everything flows from it.</span> Everything depends on it.</p>

<br>
![fernando-cejas](/assets/images/clean_architecture_reloaded_packages.png){:class="img-responsive"}
<br>

<p class="justify"><span class="underlinetext">My second tip</span> is to have a core module which will have these main responsitilities:</p>

  * <span class="boldtext">Handle global dependency injection.</span>
  * <span class="boldtext">Contain extension functions.</span>
  * <span class="boldtext">Contain the main framework abstractions.</span>
  * <span class="boldtext">Initiate in the main application common 3rd party libraries like Analytics, Crash Reporting, etc.</span>

<p class="justify"><span class="underlinetext">My third tip</span> is not at codebase level, but it might be helpful to add <span class="boldtext">code ownership</span> if we are working with feature teams, which is a win in growing organizations where many developers are working on the same codebase.</p>

<p class="justify">These are the main <span class="boldtext">benefits of Modularization:</span></p>

  * <span class="boldtext">Faster Build Time.</span>
  * <span class="boldtext">Package cohesion.</span>
  * <span class="boldtext">Re-usability of common functionality.</span>
  * <span class="boldtext">Conflicts reduction (especially when working with git flows).</span>
  * <span class="boldtext">Feature encapsulation.</span>
  * <span class="boldtext">More controlled dependendencies.</span>
  * <span class="boldtext">Team work: collaboration between teams.</span>

<p class="justify"><span class="underlinetext">I know all this sounds good on paper</span> and although modularizing your android codebase is <span class="boldtext">tricky</span> and <span class="boldtext">challenging</span> (due to all the moving parts involved), the <span class="boldtext">advantages are huge.</span></p>

<br>
# Other Implementation Details 

  * <p class="justify"><a href="https://github.com/ReactiveX/RxJava" target="_blank">RxJava:</a> <span class="boldtext">This one was one of the biggest changes</span> in comparison with the <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">previous example.</a> I got rid of <span class="boldtext">RxJava</span> because I do not need it here. By the way, I bumped into many people ONLY using it because of <span class="boldtext">threading.</span> Of course it facilitates our life in that aspect but brings <span class="boldtext">overhead</span> and <span class="boldtext">complexity</span> in other areas. <span class="boldtext">So make sure threading is not the only reason to introduce it in your codebase.</span></p>

  * <p class="justify"><span class="boldtext">Dependency Injection:</span> Simple one using <a href="https://github.com/google/dagger" target="_blank">Dagger 2,</a> and it is <span class="underlinetext">out of scope</span> of this article. <a href="https://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">I wrote about it in the past.</a></p>

  * <p class="justify"><span class="boldtext">Unit and integration tests</span> <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/tree/master/app/src/test/kotlin/com/fernandocejas/sample" target="_blank">are in place.</a></p>

  * <p class="justify"><span class="boldtext">Acceptance Tests:</span> Still <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">TODO</a> (<a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/blob/master/app/src/androidTest/kotlin/com/fernandocejas/sample/AcceptanceTest.kt" target="_blank">PRs</a> are more than welcome).</p>

<br>
# Source code and discussions

<p class="justify"><span class="boldtext">Source code can be found here:</span> <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin" target="_blank">https://github.com/android10/Android-CleanArchitecture-Kotlin</a><br>
<span class="boldtext">For discussions,</span> <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/issues" target="_blank">open a new issue on Github,</a> so we can continue the <span class="boldtext">conversation</span> <a href="https://github.com/android10/Android-CleanArchitecture-Kotlin/issues" target="_blank">over there.</a></p>

<br>
# Conclusion

<p class="justify">We have seen the <span class="boldtext">theoretical approach</span> and <span class="boldtext">implementation details.</span> There is much more to explore but that is <span class="underlinetext">homework</span> for you :).</p>

<p class="justify">Also, <span class="underlinetext">it does not matter</span> <span class="boldtext">which architecture you pick</span> for your projects as soon as it <span class="boldtext">fulfills your needs</span> and <span class="boldtext">solves your problems.</span></p>

<p class="justify">Keep in mind that there are <span class="boldtext">NO silver bullets</span> and of course there is always room for <span class="boldtext">improvement,</span> although this should be useful as a <span class="boldtext">starting point.</span></p>

<p class="justify">And some <span class="underlinetext">advice:</span></p> 

  * <span class="boldtext">Do not over think (do not do over engineering).</span>
  * <span class="boldtext">Be pragmatic.</span>
  * <span class="boldtext">Minimize framework (android) dependencies in your project as much as you can.</span>
  * <span class="boldtext">Continuous improvement through refactor.</span>
  * <span class="boldtext">Do not write code without tests (I should not be saying this in 2018 :)).</span>

<p class="justify">Something else to add since I have seen discussions around <span class="underlinetext">FP vs OOP:</span> <span class="boldtext">Functional Programming is not new and has been there for a long time</span> but nowadays is gaining more and more adopters. And the fact that Kotlin treats <span class="boldtext">functions</span> as <span class="boldtext">first class citizens</span> give us even more <span class="boldtext">power</span> and <span class="boldtext">tools</span> to <span class="underlinetext">solve our problems.</span></p>

<p class="justify"><span class="boldtext">Use whatever works for you,</span> here I'm conbining what I consider the <span class="boldtext">best of both worlds...</span></p>

<p class="justify"><a href="https://twitter.com/fernando_cejas/status/989048232805314562" target="_blank">So, my dear developer:</a></p>

![fernando-cejas](/assets/images/clean_architecture_reloaded_tweet.png){:class="img-responsive"}

# References

  * <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android...The clean way?</a>
  * <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android...The evolution.</a>
  * <a href="https://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/" target="_blank">Tasting Dagger 2 on Android.</a>
  * <a href="https://fernandocejas.com/2016/12/24/clean-architecture-dynamic-parameters-in-use-cases/" target="_blank">Clean Architecture: Dynamic Parameters in Use Cases.</a>

  * <a href="https://instagram-engineering.com/app-modularization-and-module-lazy-loading-at-instagram-and-beyond-46b9daa3fea4" target="_blank">App modularization and module lazy loading at Instagram and beyond.</a>
  * <a href="https://msdn.microsoft.com/en-us/library/hh848246.aspx" target="_blank">The MVVM Pattern.</a>
  * <a href="http://blog.xebia.com/try-option-or-either/" target="_blank">Try, Option or Either?.</a>
  * <a href="https://proandroiddev.com/kotlins-nothing-type-946de7d464fb" target="_blank">Kotlin's Nothing Type.</a>