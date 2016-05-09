title: Android, Kotlin and other JVM languages
date: 2016/05/01 12:00
authorId: JMT
tags: [kotlin, android, jvm languages]
---

<!-- more -->

Android specifics
-----------------

Although the language for writing apps is Java, Android has always been a bit different than a typical server-side Java development environment. The virtual machine (originally Dalvik; ART since KitKat) is not a regular JVM. It doesn't support the *InvokeDynamic* instruction, so dynamic languages need to resort to reflection, which is still quite slow even nowadays (although the situation is not as bad as it used to be at the time of single-core phones with 512 MB of RAM). Runtime code generation is a problem as well, because the byte code has to be written to and read from the file system which is again very slow (it can delay the app starts by several seconds).

Because of this, the obvious Java alternatives such as **Groovy** or JRuby have always struggled to be of practical use. However, Groovy's [2.4 release](http://groovy-lang.org/releasenotes/groovy-2.4.html) finally brings official Android support, such as a Gradle plugin and special *grooid* jar variants optimized for mobile usage. In order to avoid the above-mentioned performance issues, it's still recommended to use static compilation wherever possible and limit the bytecode size using ProGuard, but Groovy's expressiveness can still significantly reduce the typical Android boilerplate.

Java 8
------

Language-wise, Android is lagging behind official Java as well. The syntax-sugar features of Java 7 (such as diamond operator or switch on strings) have been available for quite some time, but any API-based language features are still dependent on the platform version supported by the given app. The *try-with-resources* block for example requires API level 19 (KitKat), but the various Jelly Bean versions still occupy about 20% of the market.

Even if we ignore the APIs that make no sense on Android such as Swing or CORBA, it is still not fully Java 7 compliant at the moment: The ForkJoin framework has been implemented in API 21, but the NIO.2 is still missing. The reason for this is that Google based the Android SDK on *Apache Harmony* - an alternative, but currently dead JRE implementation, so they have been forced to reimplement most of the the new language and library features from scratch.

But the good news is that Google and Oracle recently announced that they finally agreed on adopting OpenJDK for the Android SDK and the first results can already be seen in the [Android N preview](http://developer.android.com/preview/j8-jack.html). Thanks to the new Jack compiler, lambda expressions and method references can finally be used! They are compiled into traditional anonymous classes, so it's possible to backport them even to older platform versions in the same manner as other syntactic sugar. Default and static interface methods are supported as well, and currently, at least the whole Stream API is implemented and we can possibly expect most or all of the Java 7/8 APIs to be included in the final N release. However, it will still take a couple of years before the app developers can require N as the minimum SDK level. And in the end, even Java 8 is still only Java and we can do better than that, can't we?

The Babylon of languages
------------------------

We've already covered Groovy, so what other options are there? A popular Java alternative in the server world is **Scala**, which blends OOP with functional programming into a very powerful, yet type-safe language. Being statically-typed, it doesn't suffer as much perfomance-wise (although it relies on higher-order constructs much more than Java) and the size of the standard library (which includes a massive collection framework) isn't much of a problem anymore. The real issue can come from the complexity of using an academia-based language that supports every feature ever created. Interoperability with Java is mostly one-way and the tooling support (e.g. Gradle integration) isn't first class as well. But still, the boilerplate reduction with the flexibility and power of the language (on par with Groovy) can probably outweigh this.

Another JVM language whose popularity is on the rise, is **Clojure**. Like Groovy, it has to pay the price for dynamism, but the optimizing compiler named *Skummet* aims to address this. Clojure seems a bit like a world of its own, not only syntactically, but also from the tooling point of view, so in the end, the discussion is not as much about Android developers using Clojure, but Clojure developers writing apps for Android.

A direct competition to Kotlin is **Ceylon**, also a recent statically-typed language developed by Gavin King (the author of Hibernate) at Red Hat. One of its main features is a strong type system without the complexity of Scala. Similarly to Kotlin, it supports nullable types such as `String?`, whose value can either be a String or null (as opposed to `String` which is compile-time checked not to contain null). However, in case of Ceylon, this is just an alias of `String | Null`, which is a union type. We can know them from Java's multicatch blocks, but here, they are a first-class language feature. One of the issues with Ceylon is the tooling support: targeting Android is not its priority and the official Gradle plugin is currently only in version `0.0.2`.

Kotlin
------

So how does **Kotlin** stand in all this? According to the official website, it's a *Statically typed programming language for the JVM, Android and the browser* and also *100% interoperable with Java*.  When looking into the notes for the recent [1.0 release](https://blog.jetbrains.com/kotlin/2016/02/kotlin-1-0-released-pragmatic-language-for-jvm-and-android/), it's clear that the main idea behind the its design is **pragmatism**: Rather than offering lots of fancy and magical language features or a massive standard library, it just tackles the most painful issues with Java, such as null handling, lack of properties and the inability to add methods to 3rd party classes, while maintaining bidirectional compatibility with existing Java code and providing great tooling support as well. In the end, the language just feels like "a better Java".

One of the main selling points of Kotlin are *(not)nullable types* (but without the union types as in Ceylon), where nullness of values is explicit and null-checking is enforced by the compiler, so that the dreaded `NullPointerException` is nearly impossible to be thrown at runtime when developing in pure Kotlin. However, because we would like to be able to reuse existing libraries and frameworks written in Java (which don't have any nullness information on them), we would be forced to treat all of their APIs as nullable, even though many methods' contracts guarantee to never return null. Because of that, JetBrains have made the *pragmatic* choice to relax the restriction a bit and to allow assigning values coming from Java into not-nullable types. However, the compiler places an assertion at the place of assignment which stops the null from propagating further.

Kotlin also has language support for object *properties* and it's possible not only to use such syntax when accessing Java-based getters and setters, but even using them from the Java side is not a problem, because they get compiled as a pair of `get`/`set` methods.

Another feature implemented with Java interoperability in mind are *extension methods*. They are an easy way to add methods to existing classes that we can't modify or extend (such as 3rd party libraries). They are actually compiled into static methods where the first parameter is the type being extended, so from Java, they actually look like ordinary static utility methods. In order to do so, we define a package-level function whose name is prefixed by the name of the extended type:

```kotlin
// StringExtensions.kt

@file:JvmName("StringUtils")
package my.extensions

fun String.reverse() = StringBuilder(this).reverse().toString()
```

Calling that method from Kotlin is then as simple as:

```kotlin
import my.extensions.reverse // importing a function, not a class

"foo".reverse()
```

In Java, we import the generated class that the package function was compiled into. The default name is derived from the file name, so it would be `StringExtensionsKt` in this case, but we can use the above annotation to specify a more friendly looking name:

```java
import my.extensions.StringUtils;

StringUtils.reverse("foo");
```

Given that Kotlin is developed by JetBrains, the authors of IntelliJ IDEA and its derivates, great tooling support is no surprise. Once we download the official plugin into our Android Studio or IntelliJ, enabling Kotlin support in the project is as simple as clicking the *Configure Kotlin in Project* command in the Tools menu. This will add the Gradle plugin and the required dependencies into the build file. After that, we can start writing Kotlin classes either in the `src/main/kotlin` directory or we can put them directly into `src/main/java` along with the existing Java classes. But that's not all, the IDE even provides a convenient action that automatically converts existing Java classes into Kotlin! However, as we'll see in the following exercise, the such code sometimes requires a bit of manual fine-tuning to get it working.

{% img center-block /attachments/2016-05-11-android-kotlin/convert-to-kotlin.png %}

Code example
------------

We'll illustrate the process of converting Java to Kotlin on a very simplistic [TODO app](https://github.com/natix643/kotlin-todo). It only contains a single listview of todo items, each with a text and a checkbox to mark it's completion. Pressing the plus button opens a dialog with text input for adding a new todo. The items can be deleted either individually or it's possible to delete all completed todos using the overflow menu.

We'll start with the `master` branch, which contains only Java, and try to reach the same state as in the `kotlin` branch, which already contains a working result. The source code contains only 4 classes:
 - `Todo` - model for a todo item
 - `TodoAdapter` - list adapter which creates layout for each todo
 - `TextInputDialog` - dialog fragment with a text field
 - `MainActivity` - activity that wires all this together
 
