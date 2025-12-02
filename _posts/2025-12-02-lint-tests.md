---
layout: post
title: "Lint Me: Test Files️"
tags:
    - android
    - lint
---

It has been a year, which means it is once again time to re-examine our [TODO Lint rule]({% post_url 2024-07-22-todo-detector-v2 %}).

TL;DR: The rule checks if a `TODO` includes mandatory information -- an assignee and a date.

<figure class="align-center">
<a href="https://imgur.com/yPGU7eX"><img src="https://i.imgur.com/yPGU7eX.png" width="400" /></a>
</figure>

We have also [explored providing alternatives]({% post_url 2024-07-24-lintfix-alternatives %}) when suggesting quick fixes:

<figure class="align-center">
<a href="https://imgur.com/VTc8m4F"><img src="https://i.imgur.com/VTc8m4F.gif" /></a> 
<figcaption>A Lint rule with alternative fixes</figcaption>
</figure>

Today we will add another feature: enforcing the Lint rule in test files.

Say we have this test file with an invalid `TODO`:
```kotlin
class MainActivityTest {

    // TODO write tests
}
```

A quick and simple modification is to update the `lint` configuration in the
project's `build.gradle.kts` file with `checkTestSources`:
```kotlin
lint {
    // ...
    checkTestSources = true
}
```

From the [documentation](https://developer.android.com/reference/tools/gradle-api/8.3/null/com/android/build/api/dsl/Lint#setCheckTestSources(kotlin.Boolean)) and 
the comment in the sample DSL:
> // Normally most lint checks are not run on test sources (except the checks  
> // dedicated to looking for mistakes in unit or instrumentation tests, unless  
> // ignoreTestSources is true). You can turn on normal lint checking in all  
> // sources with the following flag, false by default

For the purposes of today's discussion, I do not want to enable _all_ 
the Lint rules to run on my test sources **_but_** I _do_ want the TODO 
Detector to.

One of the parameters in a Lint rule's `Implementation` is the `Scope`. From the 
[documentation](https://github.com/googlesamples/android-custom-lint-rules/blob/main/docs/api-guide/terminology.md.html#L92):
> `Scope` is an enum which lists various types of files that a detector may want to analyze.   
> For example, there is a scope for XML files, there is a scope for Java and Kotlin files, there is a scope for .class files, and so on.  
> Typically lint cares about which **set** of scopes apply, so most of the APIs take an `EnumSet<Scope>`, but we'll often refer to this as just “the scope” instead of the “scope set”.

Read more about `Scope`s [here](https://googlesamples.github.io/android-custom-lint-rules/api-guide.html#writingalintcheck:basics/scopes). The "various types of files" the documentation refer to are listed [here](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Scope.kt;l=1?q=scope&sq=&ss=android-studio).

To recap, this is how the current `Implementation` of the TODO Detector [looks like](https://github.com/zmdominguez/lint-rule-samples/blob/f51654a2e6f4697a58da9b4414d8c0d6eca87274/lint-checks/src/main/java/dev/zarah/lint/checks/TodoDetector.kt#L294):
```kotlin
private val IMPLEMENTATION = Implementation(
    TodoDetector::class.java,
    Scope.JAVA_FILE_SCOPE
)
```

The name `Scope.JAVA_FILE_SCOPE` confused me for a little bit -- the documentation states an `EnumSet`
is required but the name implies it is a simple `Scope`. In actual truth, [it _is_ an `EnumSet`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Scope.kt;l=267?q=scope&ss=android-studio):
```kotlin
val JAVA_FILE_SCOPE: EnumSet<Scope> = EnumSet.of(JAVA_FILE)
```

So to enable inspection of test sources, we need to add the more aptly-named `Scope.TEST_SOURCES`:
```kotlin
private val IMPLEMENTATION = Implementation(
    TodoDetector::class.java,
    EnumSet.of(Scope.JAVA_FILE, Scope.TEST_SOURCES)
)
```

After rebuilding the project to pick up the Lint rule changes, errors
in test sources should now be flagged:

<figure class="align-center">
<a href="https://imgur.com/IpoW0hE"><img src="https://i.imgur.com/IpoW0hE.png" width="400" /></a>
<figcaption>Inspected Test Source</figcaption>
</figure>

For completion, we should add a test for this new configuration. The API
has changed a bit since my previous post on unit testing Lint rules [post]({% post_url 2020-11-20-todo-test %}),
but the idea is the same -- provide a `TestFile` the unit tests can run in.
Read more about `TestFile`s [in the API Guide](https://googlesamples.github.io/android-custom-lint-rules/api-guide.html#lintcheckunittesting/testfiles).

Note that when creating a `TestFile`, there is a [constructor that takes a `to` parameter](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-tests/src/main/java/com/android/tools/lint/checks/infrastructure/LintDetectorTest.java;l=381?q=LintDetectorTest&ss=android-studio):
```java
@NonNull
public static TestFile kotlin(@NonNull String to, @NonNull @Language("kotlin") String source) {
    return TestFiles.kotlin(to, source);
}
```
where `to` is the fully qualified path of the `source`.

I haven't found any direct documentation on this, but this is exactly
what we need to indicate to Lint that a file is a test source. From one of the 
[platform Lint rules](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-tests/src/test/java/com/android/tools/lint/checks/OpenForTestingDetectorTest.kt;l=68), 
it looks like adding a `test` prefix is sufficient:
```kotlin
lint().files(
    kotlin(
        "test/test/pkg/TestClass.kt",
        """
        package test.pkg
        class TestClass {
            // TODO-Zarah Some comments
        }
    """
    ).indented()
)
```

The existing unit tests for this rule are pretty comprehensive, so for this
change I opted to only add a missing date test. If this test passes it follows
that the rule works and all other scenarios would pass as well.

As always, the updates to the Detector and the tests are available on [GitHub](https://github.com/zmdominguez/lint-rule-samples/pull/11).

---

To read my past posts on this topic, check out the posts [tagged with Lint](https://zarah.dev/tags/), some
of which are linked below:
- [Anatomy of a Lint Issue]({% post_url 2020-11-18-todo-lint %})
- [Detectors Unit Testing]({% post_url 2020-11-20-todo-test %})
- [Multi-module Rules]({% post_url 2021-10-04-multi-module-lint %})
- [Multi-module Testing]({% post_url 2021-10-05-multi-module-lint-test %})
- [Providing LintFix Alternatives]({% post_url 2024-07-24-lintfix-alternatives %})

Here are some first-party resources for Lint:
- [Lint API Guide](https://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html)
- [Lint User Guide](http://googlesamples.github.io/android-custom-lint-rules/user-guide.html)
- [Lint sample project](https://github.com/googlesamples/android-custom-lint-rules)
- [Some framework rules](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-checks/src/main/java/com/android/tools/lint/checks/)
