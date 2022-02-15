---
layout: post
title: "Multi-module Lint Rules Follow Up: Suppressions ☠️"
tags:
    - android
    - lint
---
It has been a hot minute since I posted about [writing multi-module Lint rules](https://zarah.dev/2021/10/04/multi-module-lint.html) so it's time for a follow up. Today's topic: suppressions! A quick recap of where we are:

We have [written a Lint rule](https://github.com/zmdominguez/lint-rule-samples/blob/main/lint-checks/src/main/java/dev/zarah/lint/checks/DeprecatedColorInXmlDetector.kt) that checks for usages of deprecated colours (including selectors) in XML files. The rule goes through all modules in the project looking for colours that are contained in any file with the `_deprecated` suffix in the filename. We then report usages of those colours as errors. We have also [written tests](https://github.com/zmdominguez/lint-rule-samples/blob/main/lint-checks/src/test/java/dev/zarah/lint/checks/DeprecatedColorInXmlDetectorTest.kt) for our Lint rule that cover most (all?) scenarios.

### Suppression Checks 🛡️
A key mechanism we employ in our Lint rule is calling [`getPartialResults`](https://github.com/zmdominguez/lint-rule-samples/blob/d7a78ba8c2970121127e55df7db2959e932917ff/lint-checks/src/main/java/dev/zarah/lint/checks/DeprecatedColorInXmlDetector.kt#L109) in the `afterCheckEachProject` callback. We use the returned [`PartialResults`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/PartialResult.kt) to store:
- the list of deprecated colours, and 
- the list of all colour usages

in each module (If it's a bit confusing, I highly recommend reading through the [OG post](https://zarah.dev/2021/10/04/multi-module-lint.html) and maybe things will make more sense).

The [KDoc for `getPartialResults`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Context.kt;l=446;drc=f801809cdabf506b19c1b7d19eff16a358469370) point out that suppressions are not checked at this point:
> Note that in this case, the lint infrastructure will not automatically look up the error location (since there isn't one yet) to see if the issue has been suppressed (via annotations, lint.xml and other mechanisms), so you should do this yourself, via the various `LintDriver.isSuppressed` methods.

This presents us with a great opportunity to improve our `DeprecatedColorInXml` Lint rule. We don't even want to _consider_ reporting a colour usage if our Lint rule is suppressed. Since we are parsing an XML file, we can use the [`isSuppressed()` variant that takes in an `XmlContext`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/client/api/LintDriver.kt;l=3465;drc=6a64a0c6ff08e0a34226c91a71e775d2c2699ded):
```kotlin
override fun visitAttribute(context: XmlContext, attribute: Attr) {
    // The issue is suppressed for this attribute, skip it
    val isIssueSuppressed = context.driver.isSuppressed(context, ISSUE, attribute)
    if (isIssueSuppressed) return

    // ...
}

override fun visitElement(context: XmlContext, element: Element) {
  // The issue is suppressed for this element, skip it
  val isIssueSuppressed = context.driver.isSuppressed(context, ISSUE, element)
  if (isIssueSuppressed) return

  // ...
}
```

I assume that the suppression checks can also be done in `afterCheckEachProject` but why delay when we can bail out early?

### Tests 🔬
With these updates, [suppressed Lint issues in XML files](https://developer.android.com/studio/write/lint#configuring-lint-checking-in-xml) will not be reported even if they are missing from the baseline file. We can leverage our [existing tests](https://github.com/zmdominguez/lint-rule-samples/blob/d7a78ba8c2970121127e55df7db2959e932917ff/lint-checks/src/test/java/dev/zarah/lint/checks/DeprecatedColorInXmlDetectorTest.kt) to come up with new ones.

Let's write a test for an example layout file using a deprecated colour. We provide the test with two files: one for deprecated colours and another for the layout file. When we suppress the `DeprecatedColorInXml` rule in a widget in the layout file, there should not be any reported issues.

```kotlin
@Test
fun testSuppressedDeprecatedColorInWidget() {
    lint().files(
        xml(
            "res/values/colors_deprecated.xml",
            """
            <resources>
                <color name="some_colour">#d6163e</color>
            </resources>
        """
        ).indented(),
        xml(
            "res/layout/layout.xml",
            """
            <View xmlns:android="http://schemas.android.com/apk
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content">
                <TextView android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:textColor="@color/some_colour" 
                    tools:ignore="DeprecatedColorInXml" />
            </View>
        """
        ).indented()
    )
        .testModes(TestMode.PARTIAL)
        .run()
        .expectClean()
}
```

For completeness, we can also add a test where the suppression is declared in the root element of the layout file and a deprecated colour is used in a widget (i.e. move `tools:ignore="DeprecatedColorInXml"` to the `View`).

---

As always, the rule updates and the new test cases are [in Github](https://github.com/zmdominguez/lint-rule-samples/commit/83f8cfb9345cff346f395a9c8876e153fcc24ab8). 