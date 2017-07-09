---
layout: post
title: "Killing it with Kotlin Koans: Part 2"
tags:
- kotlin
- android
---
Continuing on from [Part 1]({{ site.baseurl }}{% link _posts/2017-07-08-kotlin-koans.md %}).

__Task 6:__
One letter makes a huge difference! I'm looking at you `val` and `var`.

__Task 7:__
New line formatting in this exercise is weird.

There seems to be some minor inconsistencies in the [doco](http://kotlinlang.org/docs/reference/null-safety.html):
> When we have a nullable reference `r`, we can say "if `r` is not null, use it, otherwise use some non-null value `x`":
> `val l: Int = if (b != null) b.length else -1`

`r` and `x` are nowhere in the example.

Also, that whole section on `!!` 😂

__Task 8:__
The function to be completed being at the top of the `TODO` threw me off for a bit there. I see what you did Kotlin Koan writer!!  

__Task 9:__
What's a rational number again? 😂

__Task 10:__
Wow this stumped me a bit. Letting the IDE add the unimplemented method gives the following:
``` kotlin
Collections.sort(arrayList, object : Comparator<Int> {
        override fun compare(o1: Int?, o2: Int?): Int {
            TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
        }
    })
```

Since the parameters are marked with `?`, the compiler will not let me just do any maths on them. Okay fine, I'll do the checks. However, the solution is so much simpler! 😓

__Task 11:__
Hmm this task is kinda weird. About 97% of the solution is already provided.

__Task 12:__
If I can say one thing about Kotlin, it's that it is _convenient_.