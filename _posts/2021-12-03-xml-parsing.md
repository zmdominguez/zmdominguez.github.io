---
layout: post
title: "XML Parsing in Lint: Things Are Not What They Seem 🦹‍♀️"
tags:
    - android studio
    - lint
---
About a year ago, I wrote [about including quickfixes for Lint rules](https://zarah.dev/2020/11/19/todo-detector.html).  Quick fixes appear on the context menu when Lint flags an error and allows developers to quickly address the issue. They can be applied by clicking on the link at the bottom of the dialog or pressing ALT+ENTER (⌥ + ↩) and then choosing the fix.

<center>
    <a href="https://imgur.com/KCknsbR"><img src="https://i.imgur.com/KCknsbR.png" title="source: imgur.com" /></a>
    <small>Hovering over the error brings up a dialog with the quickfix option available</small>
</center>

<center>
    <a href="https://imgur.com/ykg8LiM"><img src="https://i.imgur.com/ykg8LiM.png" title="source: imgur.com" /></a>
    <small>Pressing ALT+ENTER on the error brings up a menu with available options</small>
</center>

The images above show the rule we have in our project for data binding expression formatting. Applying the quickfix results into putting one whitespace in between the braces and the expression itself:
```xml
android:text="@{ label }"
```

On the outset, it looks like writing this quickfix is pretty straightforward -- look for the opening brace (`{`), insert a space if needed, put back the expression, insert a space if needed, put back the closing brace (`}`). But say we have this XML layout:
<script src="https://gist.github.com/zmdominguez/afaa8b4a8577054888fb690c9bbe5f43.js"></script>
(This is an overly simplified example for illustration purposes, so keep your comments to yourself)

Notice how in line 17, the `&&` characters are escaped (`&amp;&amp;`). This is because whilst [data binding allows the usage of logical operators](https://developer.android.com/topic/libraries/data-binding/expressions#common_features) like `&&`, [some characters should always be escaped](https://www.w3.org/TR/xml/#syntax) for our XML file to be syntactically correct.

### The rule :straight_ruler:

Our Lint rule checks all attributes in an XML layout file:
```kotlin
override fun getApplicableAttributes(): Collection<String>? {
    return XmlScannerConstants.ALL
}
```

And when an attribute is encountered, our callback gets triggered:
```kotlin
override fun visitAttribute(context: XmlContext, attribute: Attr)
```

We can then check if the attribute's value is a properly formatted data binding expression:
```kotlin
// An `@` character, an optional `=` character, an opening brace `{`, a space, the expression, a space, a closing brace `}`
val validPattern = Regex("@=?\\{\\s.*\\s}")

if (isDataBindingExpression(attributeValue) && !attributeValue.matches(validPattern)) {
    // Report the issue
}
```

In our example layout file above, our Lint rule will flag that line 17 is improperly formatted when it visits the `app:visible` attribute:
```xml
app:visible="@{hasValue &amp;&amp; isFeatureOn}"
```

The [`Attr` interface](https://www.w3.org/TR/DOM-Level-3-Core/core.html#ID-637646024) inherits from the [`Node` interface](https://www.w3.org/TR/DOM-Level-3-Core/core.html#ID-1950641247), so there are some methods that we can use to figure out information about the attribute. For Lint in particular, there is a method [`getValueLocation`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/client/api/XmlParser.kt;l=137;drc=4395c75d91d34b08dc393f701cd8d82e4c3bb3aa) which we can theoretically use to get the `Location` for this node's value.

Remember, `Location` is important because it lets us tell Lint where to put the squiggly red lines to highlight the error and also the place in the file where we want to replace the incorrect value.
{: .notice--warning}

### I feel a "But..." coming :raising_hand:

Using the `Location` returned by `getValueLocation`, we expect the whole value part of the attribute to be highlighted. Let's go ahead and use it to report the issue before we build our quickfix:
```kotlin
if (isDataBindingExpression(attributeValue) && !attributeValue.matches(validPattern)) {
    // Report the issue
    context.report(
        issue = DatabindingExpressionFormatDetector.ISSUE,
        scope = attribute,
        location = context.getValueLocation(attribute),
        message = "Please put one whitespace between the braces and the expression"
    )
}
```

If we run our test for this layout file, this is what we get.
```
res/layout/layout.xml:17: Warning: Please put one whitespace between the braces and the expression [DatabindingExpressionFormat]
        app:visible="@{hasValue &amp;&amp; isFeatureOn}" />
                             ~~~~~~~~~~~~~~~~~~~~~~~~~~
0 errors, 1 warnings
```

Not quite right. :thinking: To understand what is going on, let's look at what happens in the `getValueLocation` implementation. We can see that somewhere in there, t[he `LintClient` retrieves the value of the node](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/cli/src/main/java/com/android/tools/lint/LintCliXmlParser.java;l=215;drc=1d01e75bf984568ecdf198cd35bfe05c8b0cce9f):
```java
int length = node.getValue().length();
```


Now if we go back to the [W3 documentation on `Attr`](https://www.w3.org/TR/DOM-Level-3-Core/core.html#ID-637646024):

> The following table gives some examples of the relations between the attribute value in the original document (parsed attribute), the value as exposed in the DOM, and the serialization of the value:
> 
|            Examples           |            Parsed attribute value            |              Initial `Attr.value`            |             Serialized attribute value            |   |
|:-----------------------------:|:--------------------------------------------:|:--------------------------------------------:|:-------------------------------------------------:|---|
| Character reference           | `"x&#178;=5"`                                | `"x²=5"`                                     | `"x&#178;=5"`                                     |   |
| Built-in character entity     | `"y&lt;6"`                                   | `"y<6"`                                      | `"y&lt;6"`                                        |   |
| Literal newline between       | `"x=5&#10;y=6"`                              | `"x=5 y=6"`                                  | `"x=5&#10;y=6"`                                   |   |
| Normalized newline between    | `"x=5 y=6"`                                  | `"x=5 y=6"`                                  | `"x=5 y=6"`                                       |   |
| Entity e with literal newline | `<!ENTITY e '...&#10;...'> [...]> "x=5&e;y=6"` | Dependent on Implementation and Load Options | Dependent on Implementation and Load/Save Options |   |

This is important because it says that if there are escaped characters, the resolved characters will be returned when we get the attribute's value.
```
val attrValue = node.getValue() // returns "@{hasValue && isFeatureOn}"
```

And we can actually see the effects of this. The red squiggly lines are off by eight characters:
```
&amp;&amp; -> 10 characters
&& -> 2 characters
```

### Finding the right `Location` :compass:

This means that we need to find the correct `Location` on our own.
```
// This is the contents of the whole XML file
val rawText = context.getContents() ?: return Pair(null, null)

// Find where the node bounds
val nodeStart = context.parser.getNodeStartOffset(context, attribute)
val nodeEnd = context.parser.getNodeEndOffset(context, attribute)

// The full contents of the node, including the attribute name (i.e. `app:visible="@{hasValue &amp;&amp; isFeatureOn}"`)
val rawNodeText = rawText.substring(nodeStart, nodeEnd)
```

Let's now find the offset of the start of the expression (remember we cannot rely on the length of `attribute.value` because of escaped characters). Since we are working with databinding expressions, we can rely on the syntax to find where the value starts:
```
// Find the databinding prefix used
val prefixExpression = if (rawNodeText.contains(SdkConstants.PREFIX_TWOWAY_BINDING_EXPR)) {
    SdkConstants.PREFIX_TWOWAY_BINDING_EXPR
} else SdkConstants.PREFIX_BINDING_EXPR

// First character after the opening `"` in the attribute value
val attributeValueStart = rawNodeText.indexOf(prefixExpression)
```

Now that we know where the attribute's value starts and where the whole node ends, we can finally construct an accurate `Location` for the attribute value:
```kotlin
// We cannot use `context.parser.getValueLocation()` since there may be escaped characters
// Get the Location value for the actual expression within the file (not including the quotation marks)
val attributeValueLocation = Location.create(context.file, rawText, nodeStart + attributeValueStart, nodeEnd - 1)
```

### The quickfix :woman_mechanic:

When reporting a Lint issue, we can provide our users a quickfix which enables to, uhm, quickly fix the reported issue. For this rule, if either the leading and/or trailing space in the expression is missing, our quickfix will insert those into the file.

Lint has a bunch of [available types of fixes](http://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html#addingquickfixes/availablefixes) that can help us do what we want. In this case, I opted to use `replace()` and do a straight up string replacement. Of course, as with anything Lint, there are multiple ways to do this (including using regular expressions), but I chose the path of least resistance.

We can now use the `Location` information we used earlier to write our quickfix:
```kotlin
// Get the actual databinding expression (i.e., what is between `{` and `}`)
val expressionStart = attributeValueStart + prefixExpression.length // length of the expression marker
val expressionEnd = rawNodeText.lastIndexOf("}")
val rawExpressionValue = rawNodeText.substring(expressionStart, expressionEnd)

// Formulate the replacement
val replacementText = "$prefixExpression ${rawExpressionValue.trim()} }"

// Build the quickfix
val quickfix = fix()
    .name("Fix databinding expression formatting")
    .replace()
    .range(attributeValueLocation)
    .with(replacementText)
    .build()
```


And with that, we're done! The full source code for [this Detector](https://github.com/zmdominguez/lint-rule-samples/blob/main/lint-checks/src/main/java/dev/zarah/lint/checks/DatabindingExpressionFormatDetector.kt) as well as the [tests](https://github.com/zmdominguez/lint-rule-samples/blob/main/lint-checks/src/test/java/dev/zarah/lint/checks/DatabindingExpressionFormatDetectorTest.kt) are on [Github](https://github.com/zmdominguez/lint-rule-samples).