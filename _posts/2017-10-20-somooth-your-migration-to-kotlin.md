---
id: 13
title: Smooth your migration to Kotlin
date: 2017-10-20T19:15:58+00:00
author: Fernando Cejas
description: How to smoothly migrate your existing android java codebase to kotlin. These principles can also be applied to any existing technology being introduced.  
layout: post
permalink: /2017/10/20/smooth-your-migration-to-kotlin/
categories:
  - Android
  - Development
  - Kotlin
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
<p class="justify"><span class="boldtext">This is not a new topic actually</span>, especially since <a href="https://kotlinlang.org" target="_blank">Kotlin</a> is gaining terrain in the world of programming languages in general, and especially on <span class="boldtext">Android</span>. Also, I'm not going to get into details on <span class="boldtext">what Kotlin offers</span> since <span class="italictext">there is people out there doing a great job</span> (especial mention to my friend <a href="http://antonioleiva.com" target="_blank">Antonio Leiva</a>).</p>

## Kotlin Programming Language

<p class="justify">Before starting I will just mention and give a <span class="boldtext">quick summary of the main benefits of this "young?" and modern programming language</span>:</p>

  * <span class="boldtext">Kotlin is concise.</span> The less code you write, the fewer mistakes you make.
  * <span class="boldtext">Kotlin is expressive.</span> You can express whatever you want in a shorter way (Java is verbose).
  * <span class="boldtext">Kotlin is pragmatic.</span> Straight to the point without a lot of wiring.
  * <span class="boldtext">Kotlin is android-friendly.</span> As we can see in this article ;).
  * <span class="boldtext">Kotlin is type-safe.</span> Remember the billion dollars mistake?.
  * <span class="boldtext">Kotlin is functional.</span> Functions and properties are first-class citizens.
  * <span class="boldtext">Kotlin is friendly.</span> Interoperability between Kotlin and Java works almost perfect.

## Why Kotlin in Tests?

<p class="justify"><span class="italictext">We have an android codebase</span> in our old and lovely? <span class="boldtext">Java</span> and we would love to introduce this awesome language gradually, so <span class="boldtext">why not starting with tests?</span> This way we can give it a try without affecting our main application under any circumstances, <span class="boldtext">and at the same time we get the excitement of a modern and already mature language,</span> plus some training for us and our team in preparation for the big change. Sounds good right? <span class="boldtext">Let's write some code then...</span></p>

## Getting our hands dirty

<p class="justify"><span class="boldtext">Basically, the idea is to showcase how we can test our android applications using Kotlin</span>, so as a first step we need to setup and prepare our environment by <span class="boldtext">adding Kotlin dependencies</span> in our <span class="italictext">build.gradle</span> file:</p>

```groovy
buildscript {
  repositories {
    mavenCentral()
    jcenter()
  }
  dependencies {
    classpath 'org.jetbrains.kotlin:kotlin-gradle-plugin:1.0.5-2'
  }
}

apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'

...

dependencies {
  ...
  compile "org.jetbrains.kotlin:kotlin-stdlib:1.0.6"

  ...
  testCompile 'org.jetbrains.kotlin:kotlin-stdlib:1.0.6'
  testCompile 'org.jetbrains.kotlin:kotlin-test-junit:1.0.6'
  testCompile "com.nhaarman:mockito-kotlin:1.1.0"
  testCompile 'org.amshove.kluent:kluent:1.14'
}
```

<p class="justify"><span class="boldtext">Now we need to set the dedicated directories for tests written in Kotlin</span>, this is done in our <span class="boldtext">sourceSets</span> section:</p>

```groovy
android {
  ...
  sourceSets {
    test.java.srcDirs += 'src/test/kotlin'
    androidTest.java.srcDirs += 'src/androidTest/kotlin'
  }
  ...
}
```

<p class="justify"><span class="boldtext">And as a third step we want to make sure, we do not allow accidentally (for now ;)) any Kotlin code in production by also adding this defensive lines:</span></p>

```groovy
afterEvaluate {
  android.sourceSets.all { sourceSet ->
    if (!sourceSet.name.startsWith('test') || !sourceSet.name.startsWith('androidTest')) {
      sourceSet.kotlin.setSrcDirs([])
    }
  }
}
```

<p class="justify">The <a href="https://github.com/android10/Android-KotlinInTests/blob/master/app/build.gradle" target="_blank">entire file</a> can be seen in the <a href="https://github.com/android10/Android-KotlinInTests" target="_blank">sample project on Github</a>. <span class="boldtext">And now we are able to write tests in the same way as we would do in Java.</span></p>

## JUnit Tests

<p class="justify">What I only need here is <a href="http://junit.org/junit4/" target="_blank">JUnit</a>, <a href="https://github.com/nhaarman/mockito-kotlin" target="_blank">Mockito-kotlin</a> and <a href="https://github.com/MarkusAmshove/Kluent" target="_blank">Kluent</a> (a library with really cool assertion semantics).</p>
  
<p class="justify">Let's see a simple test case for a class called <span class="boldtext">GetUserDetails.java</span> which is a <span class="boldtext">UseCase</span> (<a href="http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/" target="_blank">from a Clean Architecture approach</a>) in my application:</p>

```kotlin
class GetUserDetailsTest {

  private val USER_ID = 123

  private lateinit var getUserDetails: GetUserDetails

  private val userRepository: UserRepository = mock()
  private val threadExecutor: ThreadExecutor = mock()
  private val postExecutionThread: PostExecutionThread = mock()

  @Before
  fun setUp() {
    getUserDetails = GetUserDetails(userRepository, threadExecutor, postExecutionThread)
  }

  @Test
  fun shouldGetUserDetails() {
    getUserDetails.buildUseCaseObservable(GetUserDetails.Params.forUser(USER_ID));

    verify(userRepository).user(USER_ID)
    verifyNoMoreInteractions(userRepository)
    verifyZeroInteractions(postExecutionThread)
    verifyZeroInteractions(threadExecutor)
  }
}
```

<p class="justify"><span class="boldtext">Something to pay attention is that when we need to construct our subject under test (in the setup method), we must declare it </span>&#8220;<a href="https://kotlinlang.org/docs/reference/properties.html" target="_blank">lateinit</a>&#8220;, otherwise the compiler will complain, since properties must be initialized or be abstract. Here is another test example for a <span class="boldtext">Serializer.java</span> class in the project, where you can see the <span class="boldtext">assertions</span> mentioned above:</p>

```kotlin
class SerializerTest {

  private val JSON_RESPONSE = "{\n \"id\": 1,\n " +
                              "\"cover_url\": \"http://www.android10.org/myapi/cover_1.jpg\",\n " +
                              "\"full_name\": \"Simon Hill\",\n " +
                              "\"description\": \"Curabitur gravida nisi at nibh. In hac habitasse " +
                              "platea dictumst. Aliquam augue quam, sollicitudin vitae, consectetuer " +
                              "eget, rutrum at, lorem.\\n\\nInteger tincidunt ante vel ipsum. " +
                              "Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo " +
                              "placerat.\\n\\nPraesent blandit. Nam nulla. Integer pede justo, " +
                              "lacinia eget, tincidunt eget, tempus vel, pede.\",\n " +
                              "\"followers\": 7484,\n " +
                              "\"email\": \"jcooper@babbleset.edu\"\n}"

  private var serializer = Serializer()

  @Test
  fun shouldSerialize() {
    val userEntityOne = serializer.deserialize(JSON_RESPONSE, UserEntity::class.java)
    val jsonString = serializer.serialize(userEntityOne, UserEntity::class.java)
    val userEntityTwo = serializer.deserialize(jsonString, UserEntity::class.java)

    userEntityOne.userId shouldEqual userEntityTwo.userId
    userEntityOne.fullname shouldEqual userEntityTwo.fullname
    userEntityOne.followers shouldEqual userEntityTwo.followers
  }

  @Test
  fun shouldDesearialize() {
    val userEntity = serializer.deserialize(JSON_RESPONSE, UserEntity::class.java)

    userEntity.userId shouldEqual 1
    userEntity.fullname shouldEqual "Simon Hill"
    userEntity.followers shouldEqual 7484
  }
}
```

## Robolectric (Integration?) Tests

<p class="justify"><span class="boldtext">I created a test parent class (used for each test case) in order to encapsulate everything <a href="http://robolectric.org/" target="_blank">Robolectic</a> related,</span> thus, my tests do not depend directly on this framework. The idea is that any functionality or helper method is wrapped here (<span class="boldtext">this is a lesson learned from the past</span> where I polluted all my code with Robolectric classes, so the process of migrating to a non-backward compatible newer version was very painful).</p>

```kotlin
/**
 * Base class for Robolectric data layer tests.
 * Inherit from this class to create a test.
 */
@RunWith(RobolectricTestRunner::class)
@Config(constants = BuildConfig::class,
        application = AndroidTest.ApplicationStub::class,
        sdk = intArrayOf(21))
abstract class AndroidTest {

  fun context(): Context {
    return RuntimeEnvironment.application
  }

  fun cacheDir(): File {
    return context().cacheDir
  }

  internal class ApplicationStub : Application()
}
```

<p class="justify">And here is an example of an <span class="boldtext">integration test</span> that interacts with the android framework through our <span class="boldtext">AndroidTest.kt</span> class:</p>

```kotlin
class FileManagerTest : AndroidTest() {

  private var fileManager = FileManager()

  @After
  fun tearDown() {
    fileManager.clearDirectory(cacheDir())
  }

  @Test
  fun shouldWriteToFile() {
    val fileToWrite = createDummyFile()
    val fileContent = "content"

    fileManager.writeToFile(fileToWrite, fileContent)

    fileToWrite.exists() shouldEqualTo true
  }

  @Test
  fun shouldHaveCorrectFileContent() {
    val fileToWrite = createDummyFile()
    val fileContent = "content\n"

    fileManager.writeToFile(fileToWrite, fileContent)
    val expectedContent = fileManager.readFileContent(fileToWrite)

    expectedContent shouldEqualTo fileContent
  }

  private fun createDummyFile(): File {
    val dummyFilePath = cacheDir().path + File.separator + "dummyFile"
    return File(dummyFilePath)
  }
}
```

## Espresso Acceptance (UI?) Tests

<p class="justify"><span class="boldtext">My choice here is <a href="https://google.github.io/android-testing-support-library/docs/espresso/" target="_blank">Espresso</a> since it is backed by Google</span> and from my perspective the most stable integration test framework nowadays. The same as with Robolectric,<span class="boldtext"> I decided to create a little framework on top of it,</span> let's see how it works. All my tests depend on an <span class="boldtext">AcceptanceTest.kt</span> class:</p>

```kotlin
@LargeTest
@RunWith(AndroidJUnit4::class)
abstract class AcceptanceTest<T : Activity>(clazz: Class<T>) {

  @Rule @JvmField
  val testRule: ActivityTestRule<T> = IntentsTestRule(clazz)

  val checkThat: Matchers = Matchers()
  val events: Events = Events()
}
```

<p class="justify"><span class="boldtext">Things to keep an eye on here:</span></p>

  * <span class="boldtext">A test rule is needed within Espresso (<a href="https://developer.android.com/reference/android/support/test/rule/ActivityTestRule.html" target="_blank">from the documentation</a>):</span> This rule provides functional testing of a single activity. The activity under test will be launched before each test annotated with @Test and before methods annotated with @Before. It will be terminated after the test is completed and methods annotated with @After are finished. During the duration of the test you will be able to manipulate your Activity directly.
  * <span class="boldtext">We must annotate our &#8220;testRule&#8221; field with <a href="https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/-jvm-field/" target="_blank">@JvmField</a>:</span> this is necessary to turn this Kotlin property into a JVM field that JUnit can interpret.
  * <span class="boldtext">Matchers class:</span> A wrapper around Espresso checks.
  * <span class="boldtext">Events class:</span> Another wrapper encapsulating Espresso events.

```kotlin
class Matchers {
  fun <T : Activity> nextOpenActivityIs(clazz: Class<T>) {
    intended(IntentMatchers.hasComponent(clazz.name))
  }

  fun viewIsVisibleAndContainsText(@StringRes stringResource: Int) {
    onView(withText(stringResource)).check(matches(withEffectiveVisibility(Visibility.VISIBLE)))
  }

  fun viewContainsText(@IdRes viewId: Int, @StringRes stringResource: Int) {
    onView(withId(viewId)).check(matches(withText(stringResource)))
  }
}
```

```kotlin
class Events {
  fun clickOnView(@IdRes viewId: Int) {
    onView(withId(viewId)).perform(click())
  }
}
```

<p class="justify"><span class="boldtext">Last but not least, an example of the main activity of the project</span>, which displays a view and also opens another activity when clicking on a button. The code is pretty simple but if you need a better understanding, <a href="https://github.com/android10/Android-KotlinInTests" target="_blank">you can browse the sample code on Github</a>.</p>

```kotlin
class MainActivityTest : AcceptanceTest<MainActivity>(MainActivity::class.java) {

  @Test
  fun shouldOpenHelloWorldScreen() {
    events.clickOnView(R.id.btn_hello_world)
    checkThat.nextOpenActivityIs(HelloWorldActivity::class.java)
  }

  @Test
  fun shouldDisplayAction() {
    events.clickOnView(R.id.fab)
    checkThat.viewIsVisibleAndContainsText(R.string.action)
  }
}
```

## Running our test battery

<p class="justify"><span class="boldtext">There are neither problems nor especial configuration to run our tests from Android Studio/Intellij. </span>I also added a couple of Gradle tasks in my root <a href="https://github.com/android10/Android-KotlinInTests/blob/master/build.gradle" target="_blank">build.gradle</a> file:</p>

```grovvy
task runUnitTests(dependsOn: [':app:testDebugUnitTest']) {
  description 'Run all unit tests'
}

task runAcceptanceTests(dependsOn: [':app:connectedAndroidTest']) {
  description 'Run all acceptance tests.'
}
```

<p class="justify"><span class="boldtext">Just type this from the command line:</span></p>

```
./gradlew runUnitTests
./gradlew runAcceptanceTests
```

## Wrapping up

<p class="justify"><span class="boldtext">If at some point you were considering Kotlin, there are no longer excuses to use it in production.</span> Tests are a good starting point to introduce it, plus this will help you to have a taste of what this language has to offer. <span class="boldtext">Of course as an extra ball you get all the benefits mentioned in this article and a nice training that will prepare you and your team for the big change.</span></p>

## Repo

* <a href="https://github.com/android10/Android-KotlinInTests" target="_blank">https://github.com/android10/Android-KotlinInTests</a>

## Resources

* <a href="https://yalantis.com/blog/kotlin-vs-java-syntax/" target="_blank">https://yalantis.com/blog/kotlin-vs-java-syntax/</a>  
* <a href="https://medium.com/@sergii/using-kotlin-for-tests-in-android" target="_blank">https://medium.com/@sergii/using-kotlin-for-tests-in-android</a>  
* <a href="http://blog.greenhouseci.com/greenhouse/update/android-testing-with-kotlin/" target="_blank">http://blog.greenhouseci.com/greenhouse/update/android-testing-with-kotlin/</a>
* <a href="http://hadihariri.com/2016/10/04/Mocking-Kotlin-With-Mockito/" target="_blank">http://hadihariri.com/2016/10/04/Mocking-Kotlin-With-Mockito/</a>
* <a href="https://medium.com/@elye.project/befriending-kotlin-and-mockito-1c2e7b0ef791#.35tvarv9h" target="_blank">https://medium.com/@elye.project/befriending-kotlin-and-mockito</a>