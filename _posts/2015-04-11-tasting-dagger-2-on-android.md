---
id: 6
title: Tasting Dagger 2 on Android
date: 2015-04-11T13:23:14+00:00
author: Fernando Cejas
layout: post
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
<p class="justify"><span class="boldtext">Hey!</span> Finally I decided that was a good time to get back to the blog and share what I have dealing with for the last weeks. In this occasion I would like to talk a bit about my experience with <a href="http://google.github.io/dagger/" target="_blank">Dagger 2</a>, but first I think that really worth a quick explanation about why<span class="boldtext"> I believe that <span class="boldtext">dependency</span> injection is important and why we should definitely use it in our android applications.</span></p>

<p class="justify">By the way, I assume that you have have a basic knowledge about <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> in general, and tools like <a href="http://square.github.io/dagger/" target="_blank">Dagger</a>/<a href="https://github.com/google/guice" target="_blank">Guice</a>, otherwise I would suggest you to check some of the <a href="http://antonioleiva.com/dependency-injection-android-dagger-part-1/" target="_blank">very good tutorials out there</a>. <span class="boldtext">Let's get our hands dirty then!</span></p>

## Why dependency injection?

<p class="justify">The first <span class="boldtext"><span class="underlinetext">(and indeed most important)</span></span> thing we should know about it is that has been there for a long time and uses <a href="http://en.wikipedia.org/wiki/Inversion_of_control" target="_blank">Inversion of Control</a> principle, which basically states that <span class="boldtext">the flow of your application depends on the object graph that is built up during program execution, and such a dynamic flow is made possible by object interactions being defined through abstractions.</span> This run-time binding is achieved by mechanisms such as <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> or a service locator.</p>

<p class="justify">Said that we can get to the conclusion that <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> brings us important benefits:</p>

  * Since dependencies can be injected and configured externally we can <span class="boldtext">reuse those components.</span>
  * When injecting abstractions as collaborators, we can just <span class="boldtext">change the implementation of any object without having to make a lot of changes in our codebase</span>, since that object instantiation resides in one place isolated and decoupled.
  * Dependencies can be injected into a component: it is possible to <span class="boldtext">inject mock implementations of these dependencies</span> which makes testing easier.

<p class="justify">One thing that we will see is that we can manage the scope of our instances created, which is something really cool and from my point of view, <span class="boldtext">any object or collaborator in your app should not know anything about instances creation and lifecycle and this should be managed by our dependency injection framework.</span></p>

<img class="aligncenter wp-image-343 size-full" src="/assets/migrated/dependency_inversion1.png" alt="" width="523" height="224" srcset="/assets/migrated/dependency_inversion1.png 523w, /assets/migrated/dependency_inversion1-300x128.png 300w" sizes="(max-width: 523px) 100vw, 523px" />

## What is JSR-330?

<p class="justify">Basically <a href="https://jcp.org/en/jsr/detail?id=330" target="_blank">dependency injection for Java</a> defines a standard set of annotations (and one interface) for use on injectable classes in order to to <span class="boldtext">maximize reusability, testability and maintainability of java code.</span> Both Dagger 1 and 2 (also Guice) are based on this standard which brings consistency and an standard way to do <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a>.</p>

## Dagger 1

<p class="justify"><span class="boldtext">I will be very quick here because this version is out of the purpose of this article.</span> Anyway, <a href="http://square.github.io/dagger/" target="_blank">Dagger 1</a> has a lot to offer and I would say that nowadays is the most popular dependency injector used on Android. It has been created by <a href="https://squareup.com" target="_blank">Square</a> inspired by <a href="https://github.com/google/guice" target="_blank">Guice</a>.</p>

<p class="justify">Its fundamentals are:</p>

* <span class="boldtext">Multiple injection points: dependencies, being injected.</span>
* <span class="boldtext">Multiple bindings: dependencies, being provided.</span>
* <span class="boldtext">Multiple modules: a collection of bindings that <span class="boldtext">implement</span> a feature.</span>
* <span class="boldtext">Multiple object graphs: a collection of modules that implement a scope.</span>

<p class="justify"><span class="boldtext">Dagger 1 uses compile time to figure out bindings but also uses reflection, and although it is not used to instantiate objects, it is used for graph composition.</span> All this process <span class="underlinetext">happens at runtime</span>, where Dagger tries to figure out how everything fits together, so there is a price to pay: <span class="underlinetext">inefficiency sometimes and difficulties when debugging</span>.</p>

## Dagger 2

<p class="justify"><span class="boldtext"><a href="http://google.github.io/dagger/" target="_blank">Dagger 2</a> is a fork from Dagger 1 under heavy development by Google,</span> currently version 2.0. It was inspired by <span class="boldtext">AutoValue</span> project (<a href="https://github.com/google/auto" target="_blank">https://github.com/google/auto</a>, useful if you are tired of writing equals and hashcode methods everywhere). From the beginning, the basic idea behind Dagger 2, was to make problems solvable by using code generation, <span class="boldtext">hand written code</span>, as if we were writing all the code that creates and provides our dependencies ourselves.</p>

<p class="justify">If we compare this version with its predecessor, both are quite similar in many aspects but there are also important differences that <span class="boldtext">worth mentioning:</span></p>

* <span class="boldtext">No reflection at all: graph validation, configurations and preconditions at compile time.</span>
* <span class="boldtext">Easy debugging and fully traceable: entirely concrete call stack for provision and creation.</span>
* <span class="boldtext">More performance: according to google they gained 13% of processor performance.</span>
* <span class="boldtext">Code obfuscation: it uses method dispatch, like hand written code.</span>

<p class="justify">Of course all this cool features come with a price, which makes it <span class="boldtext">less flexible</span>: for instance, there is no dynamism due to the lack of reflection.</p>

## Diving deeper

<p class="justify"><span class="boldtext">To understand Dagger 2 it is important (and probably a bit hard in the beginning) to know about the fundamentals of <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> and the concepts of each one of these guys</span> (do not worry if you do not understand them yet, we will see examples):</p>

* <span class="boldtext">@Inject:</span> Basically with this annotation we request dependencies. In other words, you use it to tell Dagger that the annotated class or field wants to participate in dependency injection. Thus, <span class="boldtext">Dagger will construct instances of this annotated classes and satisfy their dependencies.</span>

* <span class="boldtext">@Module:</span> Modules are classes whose methods provide dependencies, so we define a class and annotate it with <span class="boldtext">@Module</span>, thus, Dagger will know where to find the dependencies in order to satisfy them when constructing class instances. <span class="boldtext">One important feature of modules is that they </span><span class="boldtext">have been designed to be partitioned and composed together (for instance we will see that in our apps we can have multiple composed modules). </span>

* <span class="boldtext">@Provide:</span> Inside modules we define methods containing this annotation which tells Dagger <span class="boldtext">how we want to construct and provide those mentioned dependencies</span>.
 
* <span class="boldtext">@Component:</span> Components basically are injectors, let's say a bridge between <span class="boldtext">@Inject</span> and <span class="boldtext">@Module</span>, which its main responsibility is to put both together. <span style="line-height: 1.5;"><span class="boldtext">They just give you instances of all the types you defined</span>, for example, we must annotate an interface with <span class="boldtext">@Component</span> and list all the <span class="boldtext">@Modules</span> </span>that will compose that component, and if any of them is missing, we get errors at compile time. All the components are aware of the scope of dependencies it provides through its modules.

* <span class="boldtext">@Scope:</span> Scopes are very useful and Dagger 2 has <span class="boldtext">has a more concrete way to do scoping through custom annotations</span>. We will see an example later, but this is a very powerful feature, because as pointed out earlier, there is no need that every object knows about how to manage its own instances. An scope example would be a class with a custom <span class="boldtext">@PerActivity</span> annotation, so this object will live as long as our Activity is alive. <span class="boldtext">In other words, we can define the granularity of your scopes (@PerFragment, @PerUser, etc).</span>

* <span class="boldtext">@Qualifier:</span> <span class="boldtext">We use this annotation when the type of class is insufficient to identify a dependency.</span> For example in the case of Android, many times we need different types of context, so we might define a qualifier annotation <span class="boldtext">"@ForApplication"</span> and <span class="boldtext">"@ForActivity"</span>, thus when injecting a context we can use those qualifiers to tell Dagger which type of context we want to be provided.

## Shut up and show me the code!

<p class="justify">I guess it is too much theory for now, so let's see <span class="boldtext">Dagger 2</span> in action, although it is a good idea to first set it up by adding the dependencies in our <span class="boldtext">build.gradle</span> file:</p>

```groovy
apply plugin: 'com.neenbedankt.android-apt'

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
}
```

<p class="justify">As you can see we are adding javax annotations, compiler, the runtime library and the <a href="https://bitbucket.org/hvisser/android-apt" target="_blank">apt plugin</a>, which is necessary, otherwise the dagger annotation processor might not work properly: <span class="boldtext">I encountered problems on Android Studio.</span></p>

## Our example

<p class="justify">A few months ago I wrote an article about <a title="Architecting Android…The clean way?" href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">how to implement uncle bob's clean architecture on Android</a>, which<span class="boldtext"> I strongly recommend to read so you get a better understanding of what we are gonna do here</span>. Back then, I faced a problem when constructing and providing dependencies of most of the objects involved in my solution, which looked something like this (check out the comments):</p>

```java
@Override void initializePresenter() {
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
}
```

<p class="justify">As you can see, the way to address this problem is to use a dependency injection framework. We basically get rid of that boilerplate code (which is unreadable and understandable): <span class="boldtext">this class must not know anything about object creation and dependency provision.</span></p>

<p class="justify"> <span class="boldtext">So how do we do it?</span> Of course we use Dagger 2 features... Let me picture the structure of <span class="boldtext">my dependency injection graph:</span>
</p>

<img class="aligncenter wp-image-347 size-full" src="/assets/migrated/composed_dagger_graph1.png" alt="" width="513" height="421" srcset="/assets/migrated/composed_dagger_graph1.png 513w, /assets/migrated/composed_dagger_graph1-300x246.png 300w" sizes="(max-width: 513px) 100vw, 513px" />

<p class="justify">Let's break down this graphic and explain its parts plus some code.</p>

<p class="justify"><span class="underlinetext"><span class="boldtext">Application Component:</span></span> A component whose lifetime is the life of the application. It injects both <span class="boldtext">AndroidApplication</span> and <span class="boldtext">BaseActivity</span> classes.</p>

```java
@Singleton // Constraints this component to one-per-application or unscoped bindings.
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {
  void inject(BaseActivity baseActivity);

  //Exposed to sub-graphs.
  Context context();
  ThreadExecutor threadExecutor();
  PostExecutionThread postExecutionThread();
  UserRepository userRepository();
}
```

<p class="justify">As you can see, I use the <span class="boldtext">@Singleton</span> annotation for this component which constraints it to <span class="underlinetext">one-per-application</span>. You might be wondering why I'm exposing the <span class="boldtext">Context</span> and the rest of the classes. <span class="boldtext">This is actually an important property of how components work in Dagger: they do not expose types from their modules unless you explicitly make them available.</span> In this case in particular I just exposed those elements to <span class="underlinetext">subgraphs</span> and if you try to remove any of them, a compilation error will be triggered.</p>

<p class="justify"><span class="underlinetext"><span class="boldtext">Application Module:</span></span> This module provides objects which will live during the application lifecycle, that is the reason why all of <span class="boldtext">@Provide</span> methods use a <span class="boldtext">@Singleton</span> scope.</p>

```java
@Module
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
}
```

<p class="justify"><span class="underlinetext"><span class="boldtext">Activity Component:</span></span> A component which will live during the <span class="underlinetext">lifetime</span> of an activity.</p>

```java
@PerActivity
@Component(dependencies = ApplicationComponent.class, modules = ActivityModule.class)
public interface ActivityComponent {
  //Exposed to sub-graphs.
  Activity activity();
}
```

<p class="justify">The <span class="boldtext"><span class="underlinetext">@PerActivity</span></span> is a custom scoping annotation <span class="boldtext">to permit objects whose lifetime should conform to the life of the activity to be memorized in the correct component.</span> I really encourage to do this as a good practice, since we get these advantages:</p>

* <span class="boldtext">The ability to inject objects where and activity is required to be constructed.</span>
* <span class="boldtext">The use of singletons on a per-activity basis.</span>
* <span class="boldtext">The global object graph is kept clear of things that can be used only in activities.</span>

<p class="justify">You can see the code below:</p>

```java
@Scope
@Retention(RUNTIME)
public @interface PerActivity {}
```

<p class="justify"><span class="boldtext"><span class="underlinetext">Activity Module:</span></span> This module exposes the activity to dependents in the graph. <span class="boldtext">The reason behind this is basically to use the activity context in a fragment, for example.</span></p>

```java
@Module
public class ActivityModule {
  private final Activity activity;

  public ActivityModule(Activity activity) {
    this.activity = activity;
  }

  @Provides @PerActivity Activity activity() {
    return this.activity;
  }
}
```

<p class="justify"><span class="boldtext"><span class="underlinetext">User Component:</span></span> A scoped <span class="boldtext">@PerActivity</span> component that extends <span class="boldtext">ActiviyComponent</span>. Basically I use it in order to injects user specific fragments. Since <span class="boldtext">ActivityModule</span> <span class="boldtext">exposes the activity to the graph</span> (as mentioned earlier), whenever an activity context is needed to satisfy a dependency, Dagger will get it from there and inject it: <span class="boldtext">there is no need to re define it in sub modules</span>.</p>

```java
@PerActivity
@Component(dependencies = ApplicationComponent.class, 
           modules = {ActivityModule.class, UserModule.class})
public interface UserComponent extends ActivityComponent {
  void inject(UserListFragment userListFragment);
  void inject(UserDetailsFragment userDetailsFragment);
}
```

<p class="justify">
  <span class="boldtext"><span class="underlinetext">User Module:</span></span> A module that provides user related collaborators. <a title="Architecting Android…The clean way?" href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Based on the example</a>, <span class="boldtext">it will provide</span> <span class="boldtext">user use cases</span> basically.
</p>

```java
@Module
public class UserModule {
  @Provides @PerActivity GetUserListUseCase provideGetUserListUseCase(GetUserListUseCaseImpl getUserListUseCase) {
    return getUserListUseCase;
  }

  @Provides @PerActivity GetUserDetailsUseCase provideGetUserDetailsUseCase(GetUserDetailsUseCaseImpl getUserDetailsUseCase) {
    return getUserDetailsUseCase;
  }
}
```

## Putting everything together

<p class="justify">Now we have our <a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">dependency injection</a> graph implementation, <span class="boldtext">how do we inject dependencies?</span> Something we need to know is that Dagger give us a bunch of options to inject dependencies:</p>

* <span class="boldtext">Constructor injection:</span> by annotating the constructor of our class with <span class="boldtext">@Inject</span>.
* <span class="boldtext">Field injection:</span> by annotating a (non private) field of our class with <span class="boldtext">@Inject.</span>
* <span class="boldtext">Method injection:</span> by annotating a method with <span class="boldtext">@Inject.</span>


<p class="justify"><span class="boldtext">This is also the order used by Dagger when binding dependencies</span> and it is important because it might happen that you have some strange behavior or even <span class="boldtext">NullPointerExceptions, which means that your dependencies might not have been initialized at the moment of the object creation.</span> This is common on Android when using field injection in <span class="boldtext">Activities</span> or <span class="boldtext">Fragments</span>, since we <span class="boldtext">do not have access to their constructors</span>.</p>

<p class="justify"><a title="Architecting Android…The clean way?" href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Getting back to our example</a>, let's see how we can inject a member to our <span class="boldtext">BaseActivity</span>. In this case we do it with a class called <span class="boldtext">Navigator</span> which is responsible for managing the navigation flow in our app:</p>

```java
public abstract class BaseActivity extends Activity {

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
}
```

<p class="justify">Since <span class="boldtext">Navigator is bound by field injection it is mandatory to be provided explicitly in our ApplicationModule using @Provide annotation</span>. Finally we initialize our component and call the <span class="boldtext">inject()</span> method in order to inject our members. We do this in the <span class="boldtext">onCreate()</span> method of our <span class="boldtext">Activity</span> by calling <span class="boldtext">getApplicationComponent()</span>. This method has been added here for reusability and its main purpose is to retrieve the <span class="boldtext">ApplicationComponent</span> which was initialized in the <span class="boldtext">Application</span> object.a</p>

<p class="justify">Let's do the same with a <a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter" target="_blank">presenter</a> in a Fragment. In this case the approach is a bit different since <span class="boldtext">we are using a per-activity scoped component</span>. So our <span class="boldtext">UserComponent</span> which will inject <span class="boldtext">UserDetailsFragment</span> will reside in our <span class="boldtext">UserDetailsActivity</span>:</p>

```java
private UserComponent userComponent;
```

<p class="justify">We have to initialize it this way in the <span class="boldtext">onCreate()</span> method of the activity:</p>

```java
private void initializeInjector() {
  this.userComponent = DaggerUserComponent.builder()
      .applicationComponent(getApplicationComponent())
      .activityModule(getActivityModule())
      .build();
}
```

<p class="justify">As you can see when Dagger processes our annotations, creates implementations of our components and rename them adding a "Dagger" prefix. <span class="boldtext">Since this is a composed component, when constructing it, we must pass in all its dependencies (both components and modules).</span> Now that our component is ready, we just make it accesible in order to satisfy the fragment dependencies:</p>

```java
@Override public UserComponent getComponent() {
  return userComponent;
}
```

<p class="justify">We bind <span class="boldtext">UserDetailsFragment</span> dependencies by getting the created component and calling the <span class="boldtext">inject()</span> method passing the <span class="boldtext">Fragment</span> as a parameter:</p>

```java
@Override public void onActivityCreated(Bundle savedInstanceState) {
  super.onActivityCreated(savedInstanceState);
  this.getComponent.inject(this);
}
```

<p class="justify"><a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">For the complete example, check the repository on github</a>. There is also some refactor happening and I can tell you that one of the main ideas (<a href="https://github.com/google/dagger/tree/master/examples" target="_blank">taken from the official examples</a>) is to <span class="boldtext">have an interface as a contract which will be implemented by every class that has a component</span>. Something like this:</p>

```java
public interface HasComponent<C> {
  C getComponent();
}
```

<p class="justify">Thus, the client (for example a <span class="boldtext">Fragment</span>) can get the component (from the <span class="boldtext">Activity</span>) and use it:</p>

```java
@SuppressWarnings("unchecked")
protected <C> C getComponent(Class<C> componentType) {
  return componentType.cast(((HasComponent<C>)getActivity()).getComponent());
}
```

<p class="justify"><span class="boldtext">The use of generics here makes mandatory to do the casting but at least is gonna fail fast whether the client cannot get a component to use</span>. Just ping me if you have any thoughts/ideas on how to solve this in a better way.</p>

## Dagger 2 code generation

<p class="justify">After having a taste of Dagger's main features, <span class="boldtext">let's see how does its job under the hood</span>. To illustrate this, we are gonna take again the <span class="boldtext">Navigator</span> class and see how it is created and injected. First let's have a look at our <span class="boldtext">DaggerApplicationComponent</span> which is an implementation of our <span class="boldtext">ApplicationComponent</span>:</p>

```java
@Generated("dagger.internal.codegen.ComponentProcessor")
public final class DaggerApplicationComponent implements ApplicationComponent {
  private Provider<Navigator> provideNavigatorProvider;
  private MembersInjector<BaseActivity> baseActivityMembersInjector;

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
}
```

<p class="justify"><span class="underlinetext"><span class="boldtext">Two important things:</span></span> the first one is that since we are gonna inject our activity, we have a members injector (which Dagger translates to <span class="boldtext">BaseActivity_MembersInjector</span>):</p>

```java
@Generated("dagger.internal.codegen.ComponentProcessor")
public final class BaseActivity_MembersInjector implements MembersInjector<BaseActivity> {
  private final MembersInjector<Activity> supertypeInjector;
  private final Provider<Navigator> navigatorProvider;

  public BaseActivity_MembersInjector(MembersInjector<Activity> supertypeInjector, Provider<Navigator> navigatorProvider) {  
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

  public static MembersInjector<BaseActivity> create(MembersInjector<Activity> supertypeInjector, Provider<Navigator> navigatorProvider) {  
      return new BaseActivity_MembersInjector(supertypeInjector, navigatorProvider);
  }
}
```

<p class="justify">Basically, <span class="boldtext">this guy contains providers for all the injectable members</span> of our <span class="boldtext">Activity</span> so when we call <span class="boldtext">inject()</span> will take the accessible fields and bind the dependencies.</p>

<p class="justify"><span class="boldtext"><span class="underlinetext">The second thing,</span></span> regarding our <span class="boldtext">DaggerApplicationComponent</span>, is that we have a <span class="boldtext">Provider&lt;Navigator&gt;</span> which is no more than interface which provides instances of our class and it is constructed by a <span class="boldtext">ScopedProvider</span> (in the <span class="boldtext">initialize()</span> method) <span class="boldtext">which will memorize the scope of the created class</span>.</p>

<p class="justify">Dagger also generated a Factory called <span class="boldtext">ApplicationModule_ProvideNavigatorFactory</span> for our <span class="boldtext">Navigator</span> which is passed as a parameter to the mentioned <span class="boldtext">ScopedProvider in order to get scoped instances of our class</span>.</p>

```java
@Generated("dagger.internal.codegen.ComponentProcessor")
public final class ApplicationModule_ProvideNavigatorFactory implements Factory<Navigator> {
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
}
```

<p class="justify">This class is actually <span class="boldtext">very simple</span>, it delegates to our <span class="boldtext">ApplicationModule</span> (which contains our <span class="boldtext">@Provide method()</span>) the creation of our <span class="boldtext">Navigator</span> class.</p>

<p class="justify"><span class="boldtext">In conclusion, this really looks like hand-written code and it is very easy to understand which makes it easy to debug.</span> There is still much to explore here and a good idea is start debugging and see how Dagger deal with dependency binding.</p>

<img class="aligncenter wp-image-335 size-large" src="/assets/migrated/debugging_dagger-1024x615.png" alt="debugging_dagger" width="640" height="384" srcset="/assets/migrated/debugging_dagger-1024x615.png 1024w, /assets/migrated/debugging_dagger-300x180.png 300w" sizes="(max-width: 640px) 100vw, 640px" />

## Testing

<p class="justify">Honestly not too much to say here: for unit tests, I do not think is necessary to create any injector so I do not use Dagger, and <span class="boldtext">by injecting mock collaborators manually works fine till now</span> but when it comes to <span class="boldtext">end-to-end integration tests,</span> Dagger could make more sense: <span class="boldtext">you can replace modules with others that provide mocks.</span> I will appreciate any experience here to add it as part of the article.</p>

## Wrapping up

<p class="justify"><span class="boldtext">So far we have had a taste on what Dagger is capable of doing</span>, but still there is a long way ahead of us, so I strongly recommend to <a href="http://google.github.io/dagger/" target="_blank">read the documentation</a>, <a href="https://www.youtube.com/watch?v=oK_XtfXPkqw" target="_blank">watch videos</a> and <a href="https://github.com/gk5885/dagger-android-sample" target="_blank">have a look at the examples</a>. <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">This was actually a small sample for learning purpose</a> and I hope you have found it useful. <span class="boldtext">Remember that any feedback is always welcome.</span></p>

## Source code:

* <span class="boldtext">Example:</span> <a href="https://github.com/android10/Android-CleanArchitecture" target="_blank">https://github.com/android10/Android-CleanArchitecture</a>

## Further reading:

  * <a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">Architecting Android..the evolution</a>
  * <a href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android..the clean way?</a>
  * <a href="https://speakerdeck.com/android10/the-mayans-lost-guide-to-rxjava-on-android" target="_blank">The Mayans Lost Guide to RxJava on Android</a>
  * <a href="https://speakerdeck.com/android10/it-is-about-philosophy-culture-of-a-good-programmer" target="_blank">It is about philosophy: Culture of a good programmer</a>

## References:

* <a title="Architecting Android…The clean way?" href="http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/" target="_blank">Architecting Android…The clean way?.</a>
* <a href="https://www.youtube.com/watch?v=oK_XtfXPkqw" target="_blank">Dagger 2, A New Type of Dependency Injection.</a> 
* <a href="https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014" target="_blank">Dependency Injection with Dagger 2.</a>
* <a href="https://publicobject.com/2014/11/15/dagger-2-has-components/" target="_blank">Dagger 2 has Components.</a>
* <a href="http://google.github.io/dagger/" target="_blank">Dagger 2 Official Documentation.</a>