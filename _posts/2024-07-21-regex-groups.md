---
layout: post
title: "I Dub Thee... Marginally better at RegE`x"
tags:
    - kotlin
    - regex
---
Being a perpetual RegEx n00b, one thing I keep on forgetting is that it is easy to get tripped up when 
extracting information from an input.

I always forget that looking for a match does _not_ really just give back just the matching values -- 
they are instead contained in `Group`s.

### Matches and Groups and all the things 💅

For example, given the sentence "Welcome to zarah.dev!", the value "zarah.dev" can be extracted by enclosing a 
pattern within parentheses:
```kotlin
val input = "Welcome to zarah.dev!"

// Capture everything (. = any character, * = multiple times) 
// after the literal phrase "Welcome to " and before the literal exclamation mark
val findPattern = """Welcome to (.*)!""".toRegex()

// Find matches in the input (!! to simplify examples)
val results = findPattern.find(input)!!

```

We _know_ that in this instance there is only one value that we care for -- `zarah.dev`. But examining the 
contents of `results`, the returned `value` is actually the same as the input AND that there are two `Group`s
contained within this `MatchResult`:
```kotlin
println("Result of find: ${results.value}") // Result of find: Welcome to zarah.dev!
println("Groups in match: ${results.groups.count()}") // Groups in match: 2
```

Looking into these further, we see that the `Group`:
- at index 0 is the full input
- at index 1 is the value captured within the parentheses 

```kotlin
results.groups.forEachIndexed { i, group ->
    println("Group index $i, value is: ${group?.value}")
}

// Group index 0, value is: Welcome to zarah.dev!
// Group index 1, value is: zarah.dev
```

I was super confused by this at first, until I realised that OF COURSE it makes sense! The whole input is 
present as the first element because it **_DOES_** match the RegEx pattern that we have. 🙈

In simple enough cases like in this example, dealing with the indices is not too bad, we just need to keep in
mind that if we want to get value of anything after the "Welcome to " phrase, we always need to look at the 
value of `group[1]`.

However, once we want to capture more and more patterns, it can get very confusing very quickly.

### Gimme All The Groups 🧮

As a quick illustration, say the input is changed to something like:
```kotlin
val input = "Welcome to <site>! My name is <owner> and I talk about <topic>."
```
and we want to retrieve the values of `site`, `owner`, and `topic`. For simplicity, we will assume that input
template always stays the same.

```kotlin
val longInput = "Welcome to zarah.dev! My name is Zarah and I talk about Android."
val sitePattern = """Welcome to (.*)! My name is (.*) and I talk about (.*)\.""".toRegex()
```

Applying this pattern to the longer input:
```kotlin
results = sitePattern.find(longInput)!!
results.groups.forEachIndexed { i, group ->
    println("Group index $i, value is: ${group?.value}")
}

// Group index 0, value is: Welcome to zarah.dev! My name is Zarah and I talk about Android.
// Group index 1, value is: zarah.dev
// Group index 2, value is: Zarah
// Group index 3, value is: Android
```

It is worth noting here that there is also a convenience method [`groupValues`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-match-result/group-values.html)
available on `MatchResult` which will basically give the same information but within a `List`:
```kotlin
println(results.groupValues)

// [Welcome to zarah.dev! My name is Zarah and I talk about Android., zarah.dev, Zarah, Android]
```

This is NOT to be confused with _another_ convenience method that omits the zeroth `Group`:
```kotlin
println(results.destructured.toList())

// [zarah.dev, Zarah, Android]
```

This is good enough if we only care about the values, but there are situations where we might want to also find
the location of each value inside the source string; such as when writing a Lint rule, for example.

### Easier RegEx 🪪

Up to this point we have been dealing with indices, but what I found easiest is referring to each extracted
value by name. And this is when [`MatchNamedGroupCollection`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-match-named-group-collection/)
comes in to save the day!

From the documentation:
> Extends `MatchGroupCollection` by introducing a way to get matched groups by name, when regex supports it.

To recap, calling `find` on a `Regex` returns a `MatchResult`, which contains a `MatchGroupCollection`s.

To use `MatchNamedGroupCollection` instead, we need to give our capturing statement a name, with the syntax
being `?<NAME>`. Applying this to our example:
```kotlin
val namedSitePattern = """Welcome to (?<site>.*)! My name is (?<owner>.*) and I talk about (?<topic>.*)\.""".toRegex()
```

To make it even easier to use, we can define these names in `val`s for easy reuse:
```kotlin
val KEY_SITE = "site"
val KEY_OWNER = "owner"
val KEY_TOPIC = "topic"
val namedSitePattern = """Welcome to (?<$KEY_SITE>.*)! My name is (?<$KEY_OWNER>.*) and I talk about (?<$KEY_TOPIC>.*)\.""".toRegex()
results = namedSitePattern.find(longInput)!!
```

And then retrieve the individual `Group`s using their names:
```kotlin
println("Site: ${results.groups[KEY_SITE]?.value}")
println("Owner: ${results.groups[KEY_OWNER]?.value}")
println("Topic: ${results.groups[KEY_TOPIC]?.value}")

// Site: zarah.dev
// Owner: Zarah
// Topic: Android
```

I learned about this when I was looking at improving the [TODO Lint rule]({% post_url 2020-11-19-todo-detector %})
and it definitely made all the `String` manipulations much easier. Keen to see how TODO Lint Rule v2 looks like? 
Stay tuned! 📻
