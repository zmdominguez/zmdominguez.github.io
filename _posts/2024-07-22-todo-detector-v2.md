---
layout: post
title: "Lint Revisit: TODO Detector v2"
tags:
    - android
    - lint
---

A few years ago, I wrote about writing [a Lint rule]({% post_url 2020-11-19-todo-detector %}) to validate the format of `TODO` comments. Whilst I find that 
Lint is still difficult to grok, I have since learnt a little bit more that I feel a revisit of this rule is
warranted.

To recap, the rule enforces that all `TODO`s must follow the format:
```kotlin
// TODO-[ASSIGNEE] (DATE_TODAY): Some comments
```

The RegEx to check if a `TODO` is valid or not is a bit loosey-goosey, and just checks for a very
generic pattern:
```kotlin
/*
 Comment should follow the pattern:
    // = Two backward slashes
    \\s+ = one or more whitespaces
    TODO- = literal "TODO-"
    \\w* = zero or more word characters
    \\s+ = one or more whitespaces
    \\( = an open parentheses
    d{8} = eight numeric characters
    \\) = a close parentheses
    : = literal ":"
    .* = zero or more of any character
*/
Regex("//\\s+TODO-\\w*\\s+\\(\\d{8}\\):.*")
```

In addition, the auto-fixes in the current version of the rule is a bit naive. For one, it assumes 
that any `TODO` does not have an assignee and the auto-fix will blindly tack on the assignee and 
today's date.

For this iteration, the rule will be expanded to:
- do separate checks for the assignee and the date,
- change the date format to `yyyy-MM-dd` to make it easier to read,
- re-use either field if it already exists,
- update tests to include `LintFix`es

I had some new tricks up my sleeve this time, including named `MatchGroup`s in RegEx (read more about that
[here]({% post_url 2024-07-21-regex-groups %})).

### Being more specific 📍

As a developer, nothing annoys me more than a very vague error message. They are unhelpful and provide
no feedback on what actually caused the error, nor steps on how to fix it.

One change in this version is to split out the checks for the assignee and the date. This allows
us to provide a more specific error message to the user:

<a href="https://imgur.com/T5I3TKZ"><img src="https://i.imgur.com/T5I3TKZ.png" width="400"></a>

[//]: # (I will discuss the implementation of each check in detail so buckle up! :seatbelt:)

First up is an update to the Regex to check for the complete pattern:

```kotlin
/*
 Comment should follow the pattern:
    .* = zero or more of any character
    TODO- = literal "TODO-"
    (?<MATCH_KEY_ASSIGNEE>[^:\(\s-]+) = assignee capturing group
        [^:\(\s-]+ = one or more of any character that is not a ":", "(", whitespace, or "-"
     = literal " "
    \( = an open parenthesis
    (?<$MATCH_KEY_DATE>20[0-9]{2}-[01][0-9]-[0-3][0-9]) = date capturing group
        20[0-9]{2}-[01][0-9]-[0-3][0-9] = accepts a four-digit year, a two-digit month, and a two-digit day
                (yes technically it will allow a month value of "00" but let's deal with that next time)
    \) = a close parenthesis
    : = literal ":"
    .* = zero or more of any character
 */
val COMPLETE_PATTERN_REGEX = """.*TODO-(?<MATCH_KEY_ASSIGNEE>[^:\(\s-]+) \((?<$MATCH_KEY_DATE>20[0-9]{2}-[01][0-9]-[0-3][0-9])\):.*""".toRegex()
```

If for one reason or another the comment does not match the pattern, a cascading set of checks are
done and any issue reported as soon as they are encountered:

```kotlin
// MISSING_DATE: Date is totally absent, or in the wrong place
var issueFound = reportDateIssue(context, comment)
if (issueFound) return

// MISSING_ASSIGNEE: Assignee is totally absent
issueFound = reportAssigneeIssue(context, comment)
if (issueFound) return

// All other issues fall through to here, like if all elements are there but in the wrong order
val incident = Incident()
    .issue(IMPROPER_FORMAT)
    .location(context.getLocation(comment))
    .message("Improper format")

// Only suggest the fix for non-block comments
// Block comments are trickier to figure out, something to implement for the future!
if (comment.sourcePsi.elementType != KtTokens.BLOCK_COMMENT) {
    incident.fix(createFix(comment))
}
context.report(incident)
```

Each kind of error is differentiated via individual `Issue` definitions [shown here](#issue-definitions).

### Reporting date issues 🗓️

Instead of a simple validation for eight consecutive digits, we now do a three-part check:
- if there is nothing enclosed in parentheses, report missing date
- if there are empty parentheses (i.e. `()`), report missing date
- if there are values in parentheses, check if it is a valid date and report if not

First, we verify if there is anything at all enclosed in parentheses:
```kotlin
// Capture everything between the first opening parenthesis and the 
// last closing parenthesis
val inParensPattern = """.*TODO.*\((?<$MATCH_KEY_DATE>[^\)]*)\).*""".toRegex()
val allInParentheses = inParensPattern.find(commentText)?.groups

// If there is nothing at all, we can conclude the date is missing
if (allInParentheses == null) {

    val incident = Incident()
        .issue(MISSING_OR_INVALID_DATE)
        .location(context.getLocation(comment))
        .message("Missing date")
    
    context.report(incident)
}
```

If the comment _does_ have parentheses, we check the value contained within via the named
capturing group `MATCH_KEY_DATE`.

```kotlin
val dateMatch = inParensMatches[MATCH_KEY_DATE]
val parensValue = requireNotNull(dateMatch).value
val message = when {
    parensValue == "" -> "Missing date"
    !isValidDate(parensValue) -> "Invalid date"
    else -> null
}
```

In the snippet above, `isValidDate` checks if the value in parentheses follows the date format required
and is within the range defined in `COMPLETE_PATTERN_REGEX`:
```kotlin
private fun isValidDate(dateString: String): Boolean {
    try {
        val providedDate = LocalDate.parse(dateString, DateTimeFormatter.ofPattern(DATE_PATTERN))
        val providedYear = providedDate.year
        return providedYear in 2024..2099
    } catch (e: DateTimeParseException) {
        return false
    }
}
```

If there is an error, we can use the `IntRange` contained in `dateMatch` to show the red squiggly lines
over the specific error, like so:

<a href="https://imgur.com/yPGU7eX"><img src="https://i.imgur.com/yPGU7eX.png" width="400" /></a>

To do this, we need to figure out the exact [`Location`](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Location.kt)
that we want to highlight. We can calculate this from two pieces of information we already know:
- the `Location` of the comment within the file
- the `IntRange` of the value in parentheses within the comment

```kotlin
val commentStartOffset = context.getLocation(comment).start?.offset ?: 0
val startLocation = commentStartOffset + dateMatch.range.first
val endLocation = commentStartOffset + dateMatch.range.last

// The actual `Location` of the date value
val dateLocation = Location.create(
    file = context.file,
    contents = context.getContents(),
    startOffset = startLocation,
    endOffset = endLocation + 1,
)

// The `Location` to highlight, including the parentheses
val problemLocation = Location.create(
    file = context.file,
    contents = context.getContents(),
    startOffset = startLocation - 1,
    endOffset = endLocation + 2,
)

// Construct the `LintFix` to put in today's date
val dateFix = LintFix.create()
    .name("Update date")
    .replace()
    .range(dateLocation)
    .with(LocalDate.now().format(DateTimeFormatter.ofPattern(DATE_PATTERN)))
    .build()
    
// Report the `Incident`
val incident = Incident()
    .issue(MISSING_OR_INVALID_DATE)
    .location(problemLocation)
    .message(message)   // Whether the date is missing or invalid
    .fix(dateFix)
context.report(incident)
```

### Reporting assignee issues 🙋‍♀️

Reporting assignee issues is largely similar to reporting date issues. We first check
if there is any assignee at all, i.e., if there is word attached to the `TODO` with a dash:

```kotlin
// Capture everything after an optional dash
// until the first open parenthesis or whitespace
val assigneePattern = """.*TODO-*(?<$MATCH_KEY_ASSIGNEE>[^:\(\s-]+).*\(.*\)""".toRegex()
```

If there is no value in `MATCH_KEY_ASSIGNEE`, the `Incident` is reported and an auto-fix suggested:
<a href="https://imgur.com/cAUifq3"><img src="https://i.imgur.com/cAUifq3.png" width="400" /></a>

Unlike the date auto-fix, however, we do not have any delimiters that would contain the assignee.
We thus need to do a bit of work to figure out what part of the current `TODO` should be replaced:
```kotlin
// Find where the word "TODO" is inside the comment, and if there is a dash present
// We want to handle comments like "// TODO- " for example
var nextCharIndex = commentText.indexOf("TODO", ignoreCase = true) + 4 // length of "TODO
if (commentText[nextCharIndex] == '-') {
    ++nextCharIndex
}

// Figure out the `Location` to be updated
val commentStartOffset = context.getLocation(comment).start?.offset ?: 0
val endLocation = commentStartOffset + nextCharIndex

val addAssigneeFix = LintFix.create()
    .name("Assign this TODO")
    .replace()
    .range(Location.create(
        file = context.file,
        contents = context.getContents(),
        startOffset = commentStartOffset,
        endOffset = endLocation
    ))
    .with("// TODO-${getUserName()}")
    .build()
    
val incident = Incident()
    .issue(MISSING_ASSIGNEE)
    .location(context.getLocation(comment))
    .message("Missing assignee")
    .fix(addAssigneeFix)
context.report(incident)
```

### Reporting disordered issues 🤹

The last type of error we want to fix are those that have all the elements in it, but are in the
incorrect order such as:

```kotlin
// TODO-Zarah: Some comments (2024-07-20)

// TODO-Zarah: (2024-07-20) Some comments
```

In this scenario, constructing the `LintFix` is a bit more involved. We grab elements from the 
current comment and reconstruct them:
```kotlin
var replacementText = "// TODO"

// We are going to manipulate the existing comment text
// Drop anything before the word "TODO"
// There may or may not be a colon, so remove that separately
var commentText = comment.text
    .substringAfter("TODO")
    .substringAfter("todo")
    .removePrefix(":")
    .trimStart()
```

Grab assignee value:
```
// Find any assignee if available and re-use it
var currentAssignee = getUserName()

if (commentText.startsWith("-")) {
    val assigneeMatches = ASSIGNEE_CAPTURE_START_REGEX.find(commentText)
    if (assigneeMatches != null) {
        val assigneeMatchGroup = requireNotNull(assigneeMatches.groups[MATCH_KEY_ASSIGNEE])
        val assigneeRange = assigneeMatchGroup.range
        commentText = commentText.removeRange(assigneeRange).trimStart().removePrefix("-")
            .removePrefix(":").trimStart()
        currentAssignee = assigneeMatchGroup.value.trim()
    }
}
replacementText += "-$currentAssignee"
```

Grab the date:
```
// Find the string enclosed in parentheses
var dateReplacementValue = LocalDate.now().format(DateTimeFormatter.ofPattern(DATE_PATTERN))

val dateMatches = DATE_CAPTURE_REGEX.find(commentText)
if (dateMatches != null) {
    val dateMatchGroup = requireNotNull(dateMatches.groups[MATCH_KEY_DATE])
    commentText = commentText.removeRange(dateMatches.groups.first()!!.range).trimStart()
    dateReplacementValue = dateMatchGroup.value
}
replacementText += " ($dateReplacementValue)"
```

Follow the convention of adding a colon:
```kotlin
// Add a colon if the remaining text does not have it yet
if (!commentText.startsWith(":")) {
    replacementText += ": "
}
```

And then do a direct replacement of the whole comment:
```kotlin
val fix = LintFix.create()
    .name(message)
    .replace()
    .text(comment.text)
    .with(replacementText)
    .build()
```

### Testing all the things 🧪

There have been so many changes in this rule, which means a whole ton of new tests! I have added a whole
bunch of test scenarios to cover the different iterations I can think of.

The tests look mostly the same as before (see [this post]({% post_url 2020-11-20-todo-test %}) for reference), other than the addition of testing the `LintFix`es. Let's take 
this `TODO` for example:
```
// TODO (2024-07-20): Some comments
```
Since we know what the comment should look like after our fix is applied, we can use that information
to construct our assertion:
```
.expectFixDiffs(
    """
        Fix for src/test/pkg/TestClass1.kt line 3: Assign this TODO:
        @@ -3 +3
        -     // TODO (2024-07-20): Some comments
        +     // TODO-$assignee (2024-07-20): Some comments
    """.trimIndent()
)
```

And that's pretty much it!

All changes for both the [detector](https://github.com/zmdominguez/lint-rule-samples/blob/f51654a2e6f4697a58da9b4414d8c0d6eca87274/lint-checks/src/main/java/dev/zarah/lint/checks/TodoDetector.kt) 
and the [tests](https://github.com/zmdominguez/lint-rule-samples/blob/f51654a2e6f4697a58da9b4414d8c0d6eca87274/lint-checks/src/test/java/dev/zarah/lint/checks/TodoDetectorTest.kt)
are in GitHub.

---

To read my past posts on this topic, check out the posts [tagged with Lint](https://zarah.dev/tags/), some
of which are linked below:
- [Anatomy of a Lint issue]({% post_url 2020-11-18-todo-lint %})
- [Detectors Unit Testing]({% post_url 2020-11-20-todo-test %})
- [Multi-module rules]({% post_url 2021-10-04-multi-module-lint %})
- [Multi-module Testing]({% post_url 2021-10-05-multi-module-lint-test %})

As always, here are some first-party resources for Lint:
- [Lint API Guide](https://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html)
- [Lint User Guide](http://googlesamples.github.io/android-custom-lint-rules/user-guide.html)
- [Lint sample project](https://github.com/googlesamples/android-custom-lint-rules)
- [Framework rules](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-checks/src/main/java/com/android/tools/lint/checks/)

---

### Issue definitions: {#issue-definitions}

```kotlin
private const val REQUIRED_FORMAT = "All TODOs must follow the format `TODO-Assignee (DATE_TODAY): Additional comments`"
val IMPROPER_FORMAT: Issue = Issue.create(
    id = "ImproperTodoFormat",
    briefDescription = "TODO has improper format",
    explanation =
    """
        $REQUIRED_FORMAT
        
        The assignee and the date are required information.
    """,
    category = Category.CORRECTNESS,
    priority = 3,
    severity = Severity.ERROR,
    implementation = IMPLEMENTATION
).setAndroidSpecific(true)
val MISSING_ASSIGNEE: Issue = Issue.create(
    id = "MissingTodoAssignee",
    briefDescription = "TODO with no assignee",
    explanation =
    """
        $REQUIRED_FORMAT
        
        Please put your name against this TODO. Assignees should be a camel-cased word, for example `ZarahDominguez`.
    """,
    category = Category.CORRECTNESS,
    priority = 3,
    severity = Severity.ERROR,
    implementation = IMPLEMENTATION
).setAndroidSpecific(true)
val MISSING_OR_INVALID_DATE: Issue = Issue.create(
    id = "MissingTodoDate",
    briefDescription = "TODO with no date",
    explanation =
    """
        $REQUIRED_FORMAT
        
        Please put today's date in the yyyy-MM-dd format enclosed in parentheses, for example `(2024-07-20)`.
    """,
    category = Category.CORRECTNESS,
    priority = 3,
    severity = Severity.ERROR,
    implementation = IMPLEMENTATION
).setAndroidSpecific(true)
```
