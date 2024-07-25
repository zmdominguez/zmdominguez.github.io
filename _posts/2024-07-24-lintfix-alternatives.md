---
layout: post
title: "Lint Revisit: Providing Alternatives 🔀"
tags:
    - android
    - lint
---

In my [previous post]({% post_url 2024-07-22-todo-detector-v2 %}), we
updated our TODO Detector to be more flexible. It is also easily extensible
so that if we want to include more parameters or perhaps add more checks,
we can follow the existing pattern and modify it.

For example, what if instead of the date being in parentheses, we want a
reference to a JIRA ticket or a GitHub issue instead. Furthermore, what if
we want to restrict these issues to a set of pre-defined project-specific
prefixes? What if we want to surface those prefixes in the auto-fix options? 
Something like this maybe?

<a href="https://imgur.com/VTc8m4F"><img src="https://i.imgur.com/VTc8m4F.gif" /></a> 

Super cool right?

### Let's make it happen 👩‍🍳

Say we only allow tickets with an `ABCD` or `XYZ` prefix like in the example above. 
We first define an enum containing these prefixes:
```kotlin
enum class VALID_TICKET_PREFIXES {
    ABCD,
    XYZ,
    ;
    companion object {
        fun allPrefixes() = VALID_TICKET_PREFIXES.entries.map { it.name }
    }
}
```

And use that to construct our RegEx pattern:
```kotlin
// Only accept valid prefixes followed by a dash and one or more numbers
val ticketPattern = VALID_TICKET_PREFIXES.allPrefixes()
    .joinToString(separator = "|") { prefix ->
        "$prefix-[0-9]+"
    }
    
val COMPLETE_PATTERN_REGEX = """.*TODO-(?<MATCH_KEY_ASSIGNEE>[^:\(\s-]+) \((?<$MATCH_KEY_TICKET>$ticketPattern)\):.*""".toRegex()
```

We can still use the same checks as we do for the date:
- check if there is anything enclosed in parentheses,
- check if the value contained in `MATCH_KEY_TICKET` starts with any of the valid prefixes

When we report the issue, we can then include the valid prefixes in the issue
explanation to help users figure out what went wrong:

<a href="https://imgur.com/I9y0Vyy"><img src="https://i.imgur.com/I9y0Vyy.png" width="500" /></a>

### Offering more help 🛟

However, to make our rule even more helpful, we can include available options in 
our `LintFix`:

<a href="https://imgur.com/jF2CIQU"><img src="https://i.imgur.com/jF2CIQU.png" width="400" /></a>

This is done by adding [`alternatives()`](https://googlesamples.github.io/android-custom-lint-rules/api-guide.md.html#addingquickfixes/combiningfixes) 
to our `LintFix`:

```kotlin
// Create a fix with alternatives
val ticketAlternatives = fix().alternatives()

VALID_TICKET_PREFIXES.allPrefixes().forEach { prefix ->
    val replacement = "$prefix-"
    
    // Create an individual fix suggesting each valid prefix
    val prefixFix = fix()
        .name("Add $prefix ticket")
        .replace()
        .range(dateLocation)
        .select("($replacement)")
        .with(replacement)
        .build()
        
    // Add this fix to our alternatives
    ticketAlternatives.add(prefixFix)
}
```

In addition to putting in the prefix, I wanted to put the cursor after the dash
to make it _even easier_ for users. This way, all that's needed to be done is
put in the actual ticket number. I cannot figure out how to do that though, so
for now the newly-added prefix would be highlighted (similar to what would happen
if you click and drag the cursor).

Selecting a bunch of text can be done using, you guessed it, `select()` which 
expects a `@RegExp`. According to the documentation:

> Sets a pattern to select; if it contains parentheses, group(1) will be selected. To just set the caret, use an empty group.

According to this I should be able to set the caret, but I cannot figure out how. I
tried searching for more documentation and in the platform rules but was, alas,
unsuccessful. Do you know how to do it? Let me know please! 🙏

Anyway, now we can use this fix when the `Incident` is reported:
```kotlin
val incident = Incident()
    .issue(MISSING_OR_INVALID_PREFIX)
    .location(problemLocation)
    .message(message)
    .fix(ticketAlternatives.build())
context.report(incident)
```

Isn't it neat? 😍

### Testing alternatives 🧪

And yes! It IS possible to test these alternatives! The syntax is similar to
how we test a `LintFix`, but repeated for each alternative provided:

```kotlin
.expectFixDiffs(
    """
         Fix for src/test/pkg/TestClass1.kt line 3: Add ABCD ticket:
         @@ -3 +3
         -     // TODO-Zarah (): Some comments
         +     // TODO-Zarah ([ABCD-]|): Some comments
         Fix for src/test/pkg/TestClass1.kt line 3: Add XYZ ticket:
         @@ -3 +3
         -     // TODO-Zarah (): Some comments
         +     // TODO-Zarah ([XYZ-]|): Some comments
    """.trimIndent()
)
```

The only weird-looking thing here is the syntax for testing the `select()` directive
we included in the fix:

```kotlin
// TODO-Zarah ([ABCD-]|): Some comments

// TODO-Zarah ([XYZ-]|): Some comments
```

What this means is that any matches to the `@RegExp` we pass into `select()` must
be enclosed between `[` and `]`. I assumed the pipe (`|`) is meant to indicate
where the caret is? Maybe? 🤔

We'll leave this here for now, unless inspiration hits me and we can spiffify this 
rule even more. 👋