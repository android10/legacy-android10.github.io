---
id: 287
title: Tasting Dagger 2 on Android
date: 2015-04-11T13:23:14+00:00
author: fcejas
layout: post
guid: http://fernandocejas.com/?p=287
permalink: /2015/04/11/tasting-dagger-2-on-android/
categories:
  - Android
  - Development
  - Java
  - Software Architecture
tags:
  - android
  - androiddev
  - architecture
  - dagger
  - dagger 2
  - dagger2
  - dependency injection
  - dependency inversion
  - developer
  - developers
  - development
  - inversion of control
  - ioc
  - patterns
  - programming
---
<p style="text-align: justify;">
  <span style="color: #000080;"><strong>Hey!</strong></span> Finally I decided that was a good time to get back to the blog and share what I have dealing with for the last weeks. In this occasion I would like to talk a bit about my experience with <a href="http://google.github.io/dagger/" target="_blank">Dagger 2</a>, but first I think that really worth a quick explanation about why<span style="color: #000080;"> <strong>I believe that <span style="color: #333399;">dependency</span> injection is important and why we should definitely use it in our android applications</strong>.</span>
</p>

<p style="text-align: justify;">
  By the way, I assume that you have have a basic knowledge about <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> in general and tools like <a href="http://square.github.io/dagger/" target="_blank">Dagger</a>/<a href="https://github.com/google/guice" target="_blank">Guice</a>, otherwise I would suggest you to check some of the <a href="http://antonioleiva.com/dependency-injection-android-dagger-part-1/" target="_blank">very good tutorials out there</a>. <span style="color: #000080;"><strong>Let&#8217;s get our hands dirty then!</strong></span>
</p>

<h3 style="text-align: justify;">
  Why dependency injection?
</h3>

<p style="text-align: justify;">
  The first <span style="color: #000080;"><strong><span style="text-decoration: underline;">(and indeed most important)</span></strong></span> thing we should know about it is that has been there for a long time and uses <a href="http://en.wikipedia.org/wiki/Inversion_of_control" target="_blank">Inversion of Control</a> principle, which basically states that <strong><span style="color: #000080;">the flow of your application depends on the object graph that is built up during program execution, and such a dynamic flow is made possible by object interactions being defined through abstractions.</span></strong> This run-time binding is achieved by mechanisms such as <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> or a service locator.
</p>

<p style="text-align: justify;">
  Said that we can get to the conclusion that <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> brings us important benefits:
</p>

<li style="text-align: justify;">
  Since dependencies can be injected and configured externally we can <span style="text-decoration: underline; color: #000080;">reuse those components.</span>
</li>
<li style="text-align: justify;">
  When injecting abstractions as collaborators, we can just <span style="text-decoration: underline;"><span style="color: #000080; text-decoration: underline;">change the implementation of any object without having to make a lot of changes in our codebase</span></span>, since that object instantiation resides in one place isolated and decoupled.
</li>
<li style="text-align: justify;">
  Dependencies can be injected into a component: it is possible to <span style="text-decoration: underline;"><span style="color: #000080; text-decoration: underline;">inject mock implementations of these dependencies</span></span> which makes testing easier.
</li>

<p style="text-align: justify;">
  One thing that we will see is that we can manage the scope of our instances created, which is something really cool and from my point of view, <strong><span style="color: #000080;">any object or collaborator in your app should not know anything about instances creation and lifecycle and this should be managed by our dependency injection framework.</span></strong>
</p>

<p style="text-align: justify;">
  <a href="http://fernandocejas.com/wp-content/uploads/2015/04/dependency_inversion1.png" target="_blank"><img class="aligncenter wp-image-343 size-full" src="http://fernandocejas.com/wp-content/uploads/2015/04/dependency_inversion1.png" alt="" width="523" height="224" srcset="http://fernandocejas.com/wp-content/uploads/2015/04/dependency_inversion1.png 523w, http://fernandocejas.com/wp-content/uploads/2015/04/dependency_inversion1-300x128.png 300w" sizes="(max-width: 523px) 100vw, 523px" /></a>
</p>

<h3 style="text-align: justify;">
  What is JSR-330?
</h3>

<p style="text-align: justify;">
  Basically <a href="https://jcp.org/en/jsr/detail?id=330" target="_blank">dependency injection for Java</a> defines a standard set of annotations (and one interface) for use on injectable classes in order to to <strong><span style="color: #000080;">maximize reusability, testability and maintainability of java code.</span></strong><br /> Both Dagger 1 and 2 (also Guice) are based on this standard which brings consistency and an standard way to do <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a>.
</p>

<h3 style="text-align: justify;">
  Dagger 1
</h3>

<p style="text-align: justify;">
  I will be very quick here because this version is out of the purpose of this article. Anyway, <a href="http://square.github.io/dagger/" target="_blank">Dagger 1</a> has a lot to offer and I would say that nowadays is the most popular dependency injector used on Android. It has been created by <a href="https://squareup.com" target="_blank">Square</a> inspired by <a href="https://github.com/google/guice" target="_blank">Guice</a>.
</p>

<p style="text-align: justify;">
  Its fundamentals are:
</p>

<ul style="text-align: justify;">
  <li>
    <span style="color: #000080;"><strong>Multiple injection points: dependencies, being injected.</strong></span>
  </li>
  <li>
    <span style="color: #000080;"><strong>Multiple bindings: dependencies, being provided.</strong></span>
  </li>
  <li>
    <span style="color: #000080;"><strong>Multiple modules: a collection of bindings that <span style="color: #000080;">implement</span> a feature.</strong></span>
  </li>
  <li>
    <span style="color: #000080;"><strong>Multiple object graphs: a collection of modules that implement a scope.</strong></span>
  </li>
</ul>

<p style="text-align: justify;">
  Dagger 1 uses compile time to figure out bindings but also uses reflection, and although it is not used to instantiate objects, it is used for graph composition. All this process <span style="text-decoration: underline;">happens at runtime</span>, where Dagger tries to figure out how everything fits together, so there is a price to pay: <span style="text-decoration: underline;">inefficiency sometimes and difficulties when debugging</span>.
</p>

<h3 style="text-align: justify;">
  Dagger 2
</h3>

<p style="text-align: justify;">
  <a href="http://google.github.io/dagger/" target="_blank">Dagger 2</a> is a fork from Dagger 1 under heavy development by Google, currently version 2.0. It was inspired by AutoValue project (<a href="https://github.com/google/auto" target="_blank">https://github.com/google/auto</a>, useful if you are tired of writing equals and hashcode methods everywhere).<br /> From the beginning, the basic idea behind Dagger 2, was to make problems solvable by using code generation, <strong><span style="color: #000080;">hand written code</span></strong>, as if we were writing all the code that creates and provides our dependencies ourselves.
</p>

<p style="text-align: justify;">
  If we compare this version with its predecessor, both are quite similar in many aspects but there are also important differences that worth mentioning:
</p>

<ul style="text-align: justify;">
  <li>
    <span style="color: #000080;"><strong>No reflection at all: graph validation, configurations and preconditions at compile time.</strong></span>
  </li>
  <li>
    <span style="color: #000080;"><strong>Easy debugging and fully traceable: entirely concrete call stack for provision and creation.</strong></span>
  </li>
  <li>
    <span style="color: #000080;"><strong>More performance: according to google they gained 13% of processor performance.</strong></span>
  </li>
  <li>
    <span style="color: #000080;"><strong>Code obfuscation: it uses method dispatch, like hand written code.</strong></span>
  </li>
</ul>

<p style="text-align: justify;">
  Of course all this cool features come with a price, which makes it <span style="text-decoration: underline;">less flexible</span>: for instance, there is no dynamism due to the lack of reflection.
</p>

<h3 style="text-align: justify;">
  Diving deeper
</h3>

<p style="text-align: justify;">
  To understand Dagger 2 it is important (and probably a bit hard in the beginning) to know about the fundamentals of <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> and the concepts of each one of these guys (do not worry if you do not understand them yet, we will see examples):
</p>

<ul style="text-align: justify;">
  <li>
    <span style="color: #000080;"><strong>@Inject:</strong></span> Basically with this annotation we request dependencies. In other words, you use it to tell Dagger that the annotated class or field wants to participate in dependency injection. Thus, Dagger will construct instances of this annotated classes and satisfy their dependencies.
  </li>
</ul>

<ul style="text-align: justify;">
  <li>
    <span style="color: #000080;"><strong>@Module:</strong></span> Modules are classes whose methods provide dependencies, so we define a class and annotate it with <span style="color: #000080;">@Module</span>, thus, Dagger will know <span style="line-height: 1.5;">where to find the dependencies in order to satisfy them when constructing class instances. <span style="text-decoration: underline;">One important feature of modules is that they </span></span><span style="line-height: 1.5;"><span style="text-decoration: underline;">have been designed to be partitioned and composed together</span> (for instance we will see that in our apps we can have multiple composed modules). </span>
  </li>
</ul>

<ul style="text-align: justify;">
  <li>
    <strong><span style="color: #000080;">@Provide:</span></strong> Inside modules we define methods containing this annotation which tells Dagger <span style="text-decoration: underline;">how we want to construct and provide those mentioned dependencies</span>.
  </li>
</ul>

<ul style="text-align: justify;">
  <li>
    <span style="color: #000080;"><strong>@Component:</strong></span> Components basically are injectors, let&#8217;s say a bridge between <span style="color: #000080;">@Inject</span> and <span style="color: #000080;">@Module</span>, which its main responsibility is to put both together. <span style="line-height: 1.5;"><span style="text-decoration: underline;">They just give you instances of all the types you defined</span>, for example, we must annotate an interface with <span style="color: #000080;">@Component</span> and list all the <span style="color: #000080;">@Modules</span> </span><span style="line-height: 1.5;">that will compose that component, and if any of them is missing, we get errors at compile time. </span><span style="line-height: 1.5;">All the components are aware of the scope of dependencies it provides through its modules. </span>
  </li>
</ul>

<ul style="text-align: justify;">
  <li>
    <span style="color: #000080;"><strong>@Scope:</strong></span> Scopes are very useful and Dagger 2 has <span style="text-decoration: underline;">has a more concrete way to do scoping through custom annotations</span>. <span style="line-height: 1.5;">We will see an example later, but this is a very powerful feature, because as pointed out earlier, there is no need that every </span><span style="line-height: 1.5;">object knows about how to manage its own instances. </span><span style="line-height: 1.5;">An scope example would be a class with a custom <span style="color: #000080;">@PerActivity</span> annotation, so this object will live as long as our Activity is alive. </span><span style="line-height: 1.5;"><span style="text-decoration: underline;">In other words, we can define the granularity of your scopes</span> (@PerFragment, @PerUser, etc). </span>
  </li>
</ul>

<ul style="text-align: justify;">
  <li>
    <strong><span style="color: #000080;">@Qualifier:</span></strong> <span style="text-decoration: underline;">We use this annotation when the type of class is insufficient to identify a dependency.</span> <span style="line-height: 1.5;">For example in the case of Android, many times we need different types of context, so we might define a qualifier annotation <strong>&#8220;@ForApplication&#8221;</strong> and <strong>&#8220;@ForActivity&#8221;</strong>, thus when </span><span style="line-height: 1.5;">injecting a context we can use those qualifiers to tell Dagger which type of context we want to be provided.</span>
  </li>
</ul>

<h3 style="text-align: justify;">
  Shut up and show me the code!
</h3>

<p style="text-align: justify;">
  I guess it is too much theory for now, so let&#8217;s see <strong><span style="color: #000080;">Dagger 2</span></strong> in action, although it is a good idea to first set it up by adding the dependencies in our <span style="color: #000080;"><strong>build.gradle</strong></span> file:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="build.gradle">apply plugin: 'com.neenbedankt.android-apt'

buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
  }
}

android {
  ...
}

dependencies {
  apt 'com.google.dagger:dagger-compiler:2.0'
  compile 'com.google.dagger:dagger:2.0'
  provided 'javax.annotation:jsr250-api:1.0' 
  
  ...
}</pre>

<p style="text-align: justify;">
  As you can see we are adding javax annotations, compiler, the runtime library and the <a href="https://bitbucket.org/hvisser/android-apt" target="_blank">apt plugin</a>, which is necessary, otherwise the dagger annotation processor might not work properly, especially <span style="text-decoration: underline;">I encountered problems on Android Studio.</span>
</p>

<h3 style="text-align: justify;">
  Our example
</h3>

<p style="text-align: justify;">
  A few months ago I wrote an article about <a title="Architecting Android…The clean way?" href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">how to implement uncle bob&#8217;s clean architecture on Android</a>, which<strong><span style="color: #000080;"> I strongly recommend to read so you get a better understanding of what we are gonna do here</span></strong>. Back then, I faced a problem when constructing and providing dependencies of most of the objects involved in my solution, which looked something like this <strong>(check out the comments)</strong>:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="UserDetailsFragment.java">@Override void initializePresenter() {
    // All this dependency initialization could have been avoided by using a
    // dependency injection framework. But in this case this is used this way for
    // LEARNING EXAMPLE PURPOSE.
    ThreadExecutor threadExecutor = JobExecutor.getInstance();
    PostExecutionThread postExecutionThread = UIThread.getInstance();

    JsonSerializer userCacheSerializer = new JsonSerializer();
    UserCache userCache = UserCacheImpl.getInstance(getActivity(), userCacheSerializer,
        FileManager.getInstance(), threadExecutor);
    UserDataStoreFactory userDataStoreFactory =
        new UserDataStoreFactory(this.getContext(), userCache);
    UserEntityDataMapper userEntityDataMapper = new UserEntityDataMapper();
    UserRepository userRepository = UserDataRepository.getInstance(userDataStoreFactory,
        userEntityDataMapper);

    GetUserDetailsUseCase getUserDetailsUseCase = new GetUserDetailsUseCaseImpl(userRepository,
        threadExecutor, postExecutionThread);
    UserModelDataMapper userModelDataMapper = new UserModelDataMapper();

    this.userDetailsPresenter =
        new UserDetailsPresenter(this, getUserDetailsUseCase, userModelDataMapper);
  }</pre>

<p style="text-align: justify;">
  As you can see, the way to address this problem is to use a dependency injection framework. We basically get rid of that boilerplate code (which is unreadable and understandable): <strong><span style="color: #000080;">this class must not know anything about object creation and dependency provision.</span></strong>
</p>

<p style="text-align: justify;">
  <strong>So how do we do it? Of course we use Dagger 2 features&#8230;</strong> Let me picture the structure of my dependency injection graph:
</p>

<p style="text-align: justify;">
  <a href="http://fernandocejas.com/wp-content/uploads/2015/04/composed_dagger_graph1.png" target="_blank"><img class="aligncenter wp-image-347 size-full" src="http://fernandocejas.com/wp-content/uploads/2015/04/composed_dagger_graph1.png" alt="" width="513" height="421" srcset="http://fernandocejas.com/wp-content/uploads/2015/04/composed_dagger_graph1.png 513w, http://fernandocejas.com/wp-content/uploads/2015/04/composed_dagger_graph1-300x246.png 300w" sizes="(max-width: 513px) 100vw, 513px" /></a>
</p>

<p style="text-align: justify;">
  Let&#8217;s break down this graphic and explain its parts plus some code.
</p>

<p style="text-align: justify;">
  <span style="text-decoration: underline;"><span style="color: #000080; text-decoration: underline;"><strong>Application Component:</strong></span></span> A component whose lifetime is the life of the application. It injects both <span style="color: #000080;"><strong>AndroidApplication</strong></span> and <span style="color: #000080;"><strong>BaseActivity</strong></span> classes.
</p>

<pre class="font-size:13 nums:false lang:java decode:true " title="ApplicationComponent.java">@Singleton // Constraints this component to one-per-application or unscoped bindings.
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {
  void inject(BaseActivity baseActivity);

  //Exposed to sub-graphs.
  Context context();
  ThreadExecutor threadExecutor();
  PostExecutionThread postExecutionThread();
  UserRepository userRepository();
}</pre>

<p style="text-align: justify;">
  As you can see, I use the <span style="color: #000080;"><strong>@Singleton</strong></span> annotation for this component which constraints it to <span style="text-decoration: underline;">one-per-application</span>. You might be wondering why I&#8217;m exposing the <span style="color: #000080;"><strong>Context</strong></span> and the rest of the classes. <strong><span style="color: #000080;">This is actually an important property of how components work in Dagger: they do not expose types from their modules unless you explicitly make them available.</span></strong> In this case in particular I just exposed those elements to <span style="text-decoration: underline;">subgraphs</span> and if you try to remove any of them, a compilation error will be triggered.
</p>

<p style="text-align: justify;">
  <span style="text-decoration: underline; color: #000080;"><strong>Application Module:</strong></span> This module provides objects which will live during the application lifecycle, that is the reason why all of <strong><span style="color: #000080;">@Provide</span></strong> methods use a <strong><span style="color: #000080;">@Singleton</span></strong> scope.
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="ApplicationModule.java">@Module
public class ApplicationModule {
  private final AndroidApplication application;

  public ApplicationModule(AndroidApplication application) {
    this.application = application;
  }

  @Provides @Singleton Context provideApplicationContext() {
    return this.application;
  }

  @Provides @Singleton Navigator provideNavigator() {
    return new Navigator();
  }

  @Provides @Singleton ThreadExecutor provideThreadExecutor(JobExecutor jobExecutor) {
    return jobExecutor;
  }

  @Provides @Singleton PostExecutionThread providePostExecutionThread(UIThread uiThread) {
    return uiThread;
  }

  @Provides @Singleton UserCache provideUserCache(UserCacheImpl userCache) {
    return userCache;
  }

  @Provides @Singleton UserRepository provideUserRepository(UserDataRepository userDataRepository) {
    return userDataRepository;
  }
}</pre>

<p style="text-align: justify;">
  <span style="text-decoration: underline;"><span style="color: #000080; text-decoration: underline;"><strong>Activity Component:</strong></span></span> A component which will live during the <span style="text-decoration: underline;">lifetime</span> of an activity.
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="ActivityComponent.java">@PerActivity
@Component(dependencies = ApplicationComponent.class, modules = ActivityModule.class)
public interface ActivityComponent {
  //Exposed to sub-graphs.
  Activity activity();
}</pre>

<p style="text-align: justify;">
  The <span style="color: #000080;"><strong><span style="text-decoration: underline;">@PerActivity</span></strong></span> is a custom scoping annotation <span style="text-decoration: underline;">to permit objects whose lifetime should conform to the life of the activity to be memorized in the correct component.</span> I really encourage to do this as a good practice, since we get these advantages:
</p>

<ul style="text-align: justify;">
  <li>
    <span style="color: #000080;"><strong>The ability to inject objects where and activity is required to be constructed.</strong></span>
  </li>
  <li>
    <span style="color: #000080;"><strong>The use of singletons on a per-activity basis.</strong></span>
  </li>
  <li>
    <span style="color: #000080;"><strong>The global object graph is kept clear of things that can be used only in activities.</strong></span>
  </li>
</ul>

<p style="text-align: justify;">
  You can see the code below:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="PerActivity.java">@Scope
@Retention(RUNTIME)
public @interface PerActivity {}</pre>

<p style="text-align: justify;">
  <span style="text-decoration: underline;"><span style="color: #000080; text-decoration: underline;"><strong>Activity Module:</strong></span></span> This module exposes the activity to dependents in the graph. The reason behind this is basically to use the activity context in a fragment for example.
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="ActivityModule.java">@Module
public class ActivityModule {
  private final Activity activity;

  public ActivityModule(Activity activity) {
    this.activity = activity;
  }

  @Provides @PerActivity Activity activity() {
    return this.activity;
  }
}</pre>

<p style="text-align: justify;">
  <span style="text-decoration: underline;"><span style="color: #000080; text-decoration: underline;"><strong>User Component:</strong></span></span> A scoped <span style="color: #000080;"><strong>@PerActivity</strong></span> component that extends <strong><span style="color: #000080;">ActiviyComponent</span></strong>. Basically I use it in order to injects user specific fragments. Since <span style="color: #000080;"><strong>ActivityModule</strong></span> <span style="text-decoration: underline;">exposes the activity to the graph</span> (as mentioned earlier), whenever an activity context is needed to satisfy a dependency, Dagger will get it from there and inject it: <span style="text-decoration: underline;">there is no need to re define it in sub modules</span>.
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="UserComponent.java">@PerActivity
@Component(dependencies = ApplicationComponent.class, modules = {ActivityModule.class, UserModule.class})
public interface UserComponent extends ActivityComponent {
  void inject(UserListFragment userListFragment);
  void inject(UserDetailsFragment userDetailsFragment);
}</pre>

<p style="text-align: justify;">
  <span style="text-decoration: underline;"><span style="color: #000080; text-decoration: underline;"><strong>User Module:</strong></span></span> A module that provides user related collaborators. <a title="Architecting Android…The clean way?" href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Based on the example</a>, <strong><span style="color: #000080;">it will provide</span> <span style="color: #000080;">user use cases</span></strong> basically.
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="UserModule.java">@Module
public class UserModule {
  @Provides @PerActivity GetUserListUseCase provideGetUserListUseCase(GetUserListUseCaseImpl getUserListUseCase) {
    return getUserListUseCase;
  }

  @Provides @PerActivity GetUserDetailsUseCase provideGetUserDetailsUseCase(GetUserDetailsUseCaseImpl getUserDetailsUseCase) {
    return getUserDetailsUseCase;
  }
}</pre>

<h3 style="text-align: justify;">
  Putting everything together
</h3>

<p style="text-align: justify;">
  Now we have our <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> graph implementation, <span style="text-decoration: underline;">how do we inject dependencies?</span> Something we need to know is that Dagger give us a bunch of options to inject dependencies:
</p>

<ol style="text-align: justify;">
  <li>
    <strong><span style="color: #000080;">Constructor injection: by annotating the constructor of our class with <span style="text-decoration: underline;">@Inject</span>.</span></strong>
  </li>
  <li>
    <strong><span style="color: #000080;">Field injection: by annotating a (non private) field of our class with <span style="text-decoration: underline;">@Inject.</span></span></strong>
  </li>
  <li>
    <strong><span style="color: #000080;">Method injection: by annotating a method with <span style="text-decoration: underline;">@Inject.</span></span></strong>
  </li>
</ol>

<p style="text-align: justify;">
  <span style="color: #000080;"><strong>This is also the order used by Dagger when binding dependencies</strong></span> and it is important because it might happen that you have some strange behavior or even <strong>NullPointerExceptions</strong>, <span style="text-decoration: underline;">which means that your dependencies might not have been initialized at the moment of the object creation.</span> This is common on Android when using field injection in <span style="color: #000080;"><strong>Activities</strong></span> or <span style="color: #000080;"><strong>Fragments</strong></span>, since we <span style="text-decoration: underline;">do not have access to their constructors</span>.
</p>

<p style="text-align: justify;">
  <a title="Architecting Android…The clean way?" href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Getting back to our example</a>, let&#8217;s see how we can inject a member to our <span style="color: #000080;"><strong>BaseActivity</strong></span>. In this case we do it with a class called <span style="color: #000080;"><strong>Navigator</strong></span> which is responsible for managing the navigation flow in our app:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="BaseActivity.java">public abstract class BaseActivity extends Activity {

  @Inject Navigator navigator;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    this.getApplicationComponent().inject(this);
  }

  protected ApplicationComponent getApplicationComponent() {
    return ((AndroidApplication)getApplication()).getApplicationComponent();
  }

  protected ActivityModule getActivityModule() {
    return new ActivityModule(this);
  }
}</pre>

<p style="text-align: justify;">
  Since <strong><span style="color: #000080;">Navigator</span></strong> <span style="text-decoration: underline;"><strong>is bound by field injection it is mandatory to be provided explicitly in our ApplicationModule using @Provide annotation</strong></span>. Finally we initialize our component and call the <strong><span style="color: #000080;">inject()</span></strong> method in order to inject our members. We do this in the <strong><span style="color: #000080;">onCreate()</span></strong> method of our <strong><span style="color: #000080;">Activity</span></strong> by calling <span style="color: #000080;"><strong>getApplicationComponent()</strong></span>. This method has been added here for reusability and its main purpose is to retrieve the <span style="color: #000080;"><strong>ApplicationComponent</strong></span> which was initialized in the <strong><span style="color: #000080;">Application</span></strong> object.
</p>

<p style="text-align: justify;">
  Let&#8217;s do the same with a <a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter" target="_blank">presenter</a> in a Fragment. In this case the approach is a bit different since <span style="color: #000080;"><strong>we are using a per-activity scoped component</strong></span>. So our <strong><span style="color: #000080;">UserComponent</span></strong> which will inject <strong><span style="color: #000080;">UserDetailsFragment</span></strong> will reside in our <strong><span style="color: #000080;">UserDetailsActivity</span></strong>:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="UserDetailsActivity.java">private UserComponent userComponent;</pre>

<p style="text-align: justify;">
  We have to initialize it this way in the <span style="color: #000080;"><strong>onCreate()</strong></span> method of the activity:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="UserDetailsActivity.java">private void initializeInjector() {
  this.userComponent = DaggerUserComponent.builder()
      .applicationComponent(getApplicationComponent())
      .activityModule(getActivityModule())
      .build();
}</pre>

<p style="text-align: justify;">
  As you can see when Dagger processes our annotations, creates implementations of our components and rename them adding a &#8220;Dagger&#8221; prefix. <strong><span style="color: #000080;">Since this is a composed component, when constructing it, we must pass in all its dependencies (both components and modules).</span></strong> Now that our component is ready, we just make it accesible in order to satisfy the fragment dependencies:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="UserDetailsActivity.java">@Override public UserComponent getComponent() {
  return userComponent;
}</pre>

<p style="text-align: justify;">
  We bind <strong><span style="color: #000080;">UserDetailsFragment</span></strong> dependencies by getting the created component and calling the <strong><span style="color: #000080;">inject()</span></strong> method passing the <strong><span style="color: #000080;">Fragment</span></strong> as a parameter:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="UserDetailsFragment.java">@Override public void onActivityCreated(Bundle savedInstanceState) {
  super.onActivityCreated(savedInstanceState);
  this.getComponent.inject(this);
}</pre>

<p style="text-align: justify;">
  <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">For the complete example, check the repository on github</a>. There is also some refactor happening and I can tell you that one of the main ideas (<a href="https://github.com/google/dagger/tree/master/examples" target="_blank">taken from the official examples</a>) is to <span style="color: #000080;"><strong>have an interface as a contract which will be implemented by every class that has a component</strong></span>. Something like this:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="HasComponent.java">public interface HasComponent&lt;C&gt; {
  C getComponent();
}</pre>

<p style="text-align: justify;">
  Thus, the client (for example a <span style="color: #000080;"><strong>Fragment</strong></span>) can get the component (from the <span style="color: #000080;"><strong>Activity</strong></span>) and use it:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="BaseFragment.java">@SuppressWarnings("unchecked")
protected &lt;C&gt; C getComponent(Class&lt;C&gt; componentType) {
  return componentType.cast(((HasComponent&lt;C&gt;)getActivity()).getComponent());
}</pre>

<p style="text-align: justify;">
  <span style="color: #000080;"><strong>The use of generics here makes mandatory to do the casting but at least is gonna fail fast whether the client cannot get a component to use</strong></span>. Just ping me if you have any thoughts/ideas on how to solve this in a better way.
</p>

<h3 style="text-align: justify;">
  Dagger 2 code generation
</h3>

<p style="text-align: justify;">
  After having a taste of Dagger&#8217;s main features, <span style="text-decoration: underline;"><strong>let&#8217;s see how does its job under the hood</strong></span>. To illustrate this, we are gonna take again the <span style="color: #000080;"><strong>Navigator</strong></span> class and see how it is created and injected.<br /> First let&#8217;s have a look at our <span style="color: #000080;"><strong>DaggerApplicationComponent</strong></span> which is an implementation of our <span style="color: #000080;"><strong>ApplicationComponent</strong></span>:
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="DaggerApplicationComponent.java">@Generated("dagger.internal.codegen.ComponentProcessor")
public final class DaggerApplicationComponent implements ApplicationComponent {
  private Provider&lt;Navigator&gt; provideNavigatorProvider;
  private MembersInjector&lt;BaseActivity&gt; baseActivityMembersInjector;

  private DaggerApplicationComponent(Builder builder) {  
    assert builder != null;
    initialize(builder);
  }

  public static Builder builder() {  
    return new Builder();
  }

  private void initialize(final Builder builder) {  
    this.provideNavigatorProvider = ScopedProvider.create(ApplicationModule_ProvideNavigatorFactory.create(builder.applicationModule));
    this.baseActivityMembersInjector = BaseActivity_MembersInjector.create((MembersInjector) MembersInjectors.noOp(), provideNavigatorProvider);
  }

  @Override
  public void inject(BaseActivity baseActivity) {  
    baseActivityMembersInjector.injectMembers(baseActivity);
  }

  public static final class Builder {
    private ApplicationModule applicationModule;
  
    private Builder() {  
    }
  
    public ApplicationComponent build() {  
      if (applicationModule == null) {
        throw new IllegalStateException("applicationModule must be set");
      }
      return new DaggerApplicationComponent(this);
    }
  
    public Builder applicationModule(ApplicationModule applicationModule) {  
      if (applicationModule == null) {
        throw new NullPointerException("applicationModule");
      }
      this.applicationModule = applicationModule;
      return this;
    }
  }
}</pre>

<p style="text-align: justify;">
  <span style="text-decoration: underline;"><span style="color: #000080; text-decoration: underline;"><strong>Two important things:</strong></span></span> the first one is that since we are gonna inject our activity, we have a members injector (which Dagger translates to <strong><span style="color: #000080;">BaseActivity_MembersInjector</span></strong>):
</p>

<pre class="font-size:13 nums:false lang:java decode:true" title="BaseActivity_MembersInjector.java">@Generated("dagger.internal.codegen.ComponentProcessor")
public final class BaseActivity_MembersInjector implements MembersInjector&lt;BaseActivity&gt; {
  private final MembersInjector&lt;Activity&gt; supertypeInjector;
  private final Provider&lt;Navigator&gt; navigatorProvider;

  public BaseActivity_MembersInjector(MembersInjector&lt;Activity&gt; supertypeInjector, Provider&lt;Navigator&gt; navigatorProvider) {  
    assert supertypeInjector != null;
    this.supertypeInjector = supertypeInjector;
    assert navigatorProvider != null;
    this.navigatorProvider = navigatorProvider;
  }

  @Override
  public void injectMembers(BaseActivity instance) {  
    if (instance == null) {
      throw new NullPointerException("Cannot inject members into a null reference");
    }
    supertypeInjector.injectMembers(instance);
    instance.navigator = navigatorProvider.get();
  }

  public static MembersInjector&lt;BaseActivity&gt; create(MembersInjector&lt;Activity&gt; supertypeInjector, Provider&lt;Navigator&gt; navigatorProvider) {  
      return new BaseActivity_MembersInjector(supertypeInjector, navigatorProvider);
  }
}</pre>

<p style="text-align: justify;">
  Basically, <span style="text-decoration: underline;">this guy contains providers for all the injectable members</span> of our <span style="color: #000080;"><strong>Activity</strong></span> so when we call <strong><span style="color: #000080;">inject()</span></strong> will take the accessible fields and bind the dependencies.
</p>

<p style="text-align: justify;">
  <span style="text-decoration: underline;"><span style="color: #000080; text-decoration: underline;"><strong>The second thing,</strong></span></span> regarding our <span style="color: #000080;"><strong>DaggerApplicationComponent</strong></span>, is that we have a <strong><span style="color: #000080;">Provider<Navigator></span></strong> which is no more than interface which provides instances of our class and it is constructed by a <span style="color: #000080;"><strong>ScopedProvider</strong></span> (in the <strong><span style="color: #000080;">initialize()</span></strong> method) <span style="text-decoration: underline;">which will memorize the scope of the created class</span>.
</p>

<p style="text-align: justify;">
  Dagger also generated a Factory called <span style="color: #000080;"><strong>ApplicationModule_ProvideNavigatorFactory</strong></span> for our <strong><span style="color: #000080;">Navigator</span></strong> which is passed as a parameter to the mentioned <strong><span style="color: #000080;">ScopedProvider</span></strong> <span style="text-decoration: underline;">in order to get scoped instances of our class</span>.
</p>

<pre class="font-size:13 nums:false lang:java decode:true " title="ApplicationModule_ProvideNavigatorFactory.java">@Generated("dagger.internal.codegen.ComponentProcessor")
public final class ApplicationModule_ProvideNavigatorFactory implements Factory&lt;Navigator&gt; {
  private final ApplicationModule module;

  public ApplicationModule_ProvideNavigatorFactory(ApplicationModule module) {  
    assert module != null;
    this.module = module;
  }

  @Override
  public Navigator get() {  
    Navigator provided = module.provideNavigator();
    if (provided == null) {
      throw new NullPointerException("Cannot return null from a non-@Nullable @Provides method");
    }
    return provided;
  }

  public static Factory&lt;Navigator&gt; create(ApplicationModule module) {  
    return new ApplicationModule_ProvideNavigatorFactory(module);
  }
}</pre>

<p style="text-align: justify;">
  This class is actually <strong><span style="color: #000080;">very simple</span></strong>, it delegates to our <span style="color: #000080;"><strong>ApplicationModule</strong></span> (which contains our <strong><span style="color: #000080;">@Provide method()</span></strong>) the creation of our <strong><span style="color: #000080;">Navigator</span></strong> class.
</p>

<p style="text-align: justify;">
  <strong><span style="color: #000080;">In conclusion, this really looks like hand-written code and it is very easy to understand which makes it easy to debug.</span></strong> There is still much to explore here and a good idea is start debugging and see how Dagger deal with dependency binding.
</p>

<p style="text-align: justify;">
  <a href="http://fernandocejas.com/wp-content/uploads/2015/04/debugging_dagger.png" target="_blank"><img class="aligncenter wp-image-335 size-large" src="http://fernandocejas.com/wp-content/uploads/2015/04/debugging_dagger-1024x615.png" alt="debugging_dagger" width="640" height="384" srcset="http://fernandocejas.com/wp-content/uploads/2015/04/debugging_dagger-1024x615.png 1024w, http://fernandocejas.com/wp-content/uploads/2015/04/debugging_dagger-300x180.png 300w" sizes="(max-width: 640px) 100vw, 640px" /></a>
</p>

<h3 style="text-align: justify;">
  Testing
</h3>

<p style="text-align: justify;">
  Honestly not too much to say here: for unit tests, I do not think is necessary to create any injector so I do not use Dagger, and <span style="color: #000080;"><strong>by injecting mock collaborators manually works fine till now</strong></span> but when it comes to <strong><span style="color: #000080;">end-to-end integration tests,</span></strong> Dagger could make more sense: <span style="color: #000080;"><strong>you can replace modules with others that provide mocks.</strong></span><br /> I will appreciate any experience here to add it as part of the article.
</p>

<h3 style="text-align: justify;">
  Wrapping up
</h3>

<p style="text-align: justify;">
  <span style="color: #000080;"><strong>So far we have had a taste on what Dagger is capable of doing</strong></span>, but still there is a long way ahead of us, so I strongly recommend to <a href="http://google.github.io/dagger/" target="_blank">read the documentation</a>, <a href="https://www.youtube.com/watch?v=oK_XtfXPkqw" target="_blank">watch videos</a> and <a href="https://github.com/gk5885/dagger-android-sample" target="_blank">have a look at the examples</a>. <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">This was actually a small sample for learning purpose</a> and I hope you have found it useful. <strong><span style="color: #000080;">Remember that any feedback is always welcome.</span></strong>
</p>

<h3 style="text-align: justify;">
  Source code:
</h3>

<p style="text-align: justify;">
  Example: <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">https://github.com/android10/Android-CleanArchitecture</a>
</p>

### Further reading:

  1. <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android..the evolution</a>
  2. <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android..the clean way?</a>
  3. <a href="https://speakerdeck.com/android10/the-mayans-lost-guide-to-rxjava-on-android" target="_blank">The Mayans Lost Guide to RxJava on Android</a>
  4. <a href="https://speakerdeck.com/android10/it-is-about-philosophy-culture-of-a-good-programmer" target="_blank">It is about philosophy: Culture of a good programmer</a>

<h3 style="text-align: justify;">
  References:
</h3>

<p style="text-align: justify;">
  <a title="Architecting Android…The clean way?" href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android…The clean way?.</a><br /> <a href="https://www.youtube.com/watch?v=oK_XtfXPkqw" target="_blank">Dagger 2, A New Type of Dependency Injection.</a><br /> <a href="https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014" target="_blank">Dependency Injection with Dagger 2.</a><br /> <a href="https://publicobject.com/2014/11/15/dagger-2-has-components/" target="_blank">Dagger 2 has Components.</a><br /> <a href="http://google.github.io/dagger/" target="_blank">Dagger 2 Official Documentation.</a>
</p>