---
layout: post
title: "Multi-module Lint Rules: Tests 🧪"
tags:
    - android
    - lint
---

In my [previous post](https://zarah.dev/2021/10/04/multi-module-lint.html), I talked about how to write a Lint rule that gathers information from different modules before performing a final analysis to determine if there are errors.

Writing tests for files that reside in the same module generally follow the same principles as outlined in my post [Enforcing Team Rules with Lint: Tests](https://zarah.dev/2020/11/20/todo-test.html) so for now we will be focusing on writing tests for a multi-module setup.

We will write a test with the following set-up:
- two modules, a library and an app module
- the library module holds a `colors_deprecated.xml` file
- the app module holds a layout file that uses one of the deprecated colours


Let's set up the `colors_deprecated.xml` file:
```kotlin
val DEPRECATED_COLOUR_FILE: TestFile = xml(
    "res/values/colors_deprecated.xml",
    """
        <resources>
            <color name="red_error">#d6163e</color>
            <color name="another_value">#d6163e</color>
            <color name="and_another_value">#d6163e</color>
            </resources>
    """
).indented()
```

And the layout file consuming the colour:
```kotlin
val VIEW_WITH_DEPRECATED_COLOUR: TestFile = xml(
    "res/layout/layout.xml",
    """
        <View xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/red_error" />
    """
).indented()
```

Next we need to set up our projects (aka modules). Lint's mechanism for setting modules up in tests is via [`ProjectDescription`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-tests/src/main/java/com/android/tools/lint/checks/infrastructure/ProjectDescription.kt) (oddly enough this is missing in the API guide as well :crying_cat_face: ). Let's set up the library module first:
```kotlin
val deprecatedModule = ProjectDescription()
    .name("deprecated-library")
    .type(ProjectDescription.Type.LIBRARY)
    .files(DEPRECATED_COLOUR_FILE)
```
Here we provide the module name, declare it as a library, and include the files that should be in the module.

Let's then set up our app module:
```kotlin
val appModule = ProjectDescription()
    .name("app")
    .files(VIEW_WITH_DEPRECATED_COLOUR)
    .dependsOn(deprecatedModule)
```
Note that we call `dependsOn` here to link the two modules together.

All that's left to do now is to write the bit that runs the test:
```kotlin
lint()
    // Add modules to the test task
    .projects(deprecatedModule, appModule)
    // Tells Lint to analyse all modules first before reporting issues
    .testModes(TestMode.PARTIAL)
    .run()
    .expect(
        """
        res/layout/layout.xml:4: Error: Deprecated colours should not be used [DeprecatedColorInXml]
            android:background="@color/red_error" />
                                ~~~~~~~~~~~~~~~~
            1 errors, 0 warnings
        """
    )

```

Using the same principles we can write tests for analysing transitive dependencies as well. For the full test suite for this Lint rule, check out the source code on [Github](https://github.com/zmdominguez/lint-rule-samples/blob/main/lint-checks/src/test/java/dev/zarah/lint/checks/DeprecatedColorInXmlDetectorTest.kt).