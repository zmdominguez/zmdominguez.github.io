---
layout: post
title: String Pluralization
date: '2010-09-15T13:04:00.007+10:00'
author: Zarah Dominguez
tags:
- plurals
- strings
- android
modified_time: '2010-09-16T01:09:56.913+10:00'
thumbnail: http://2.bp.blogspot.com/_p3DeelS9Gp8/TJDhPBxbZ2I/AAAAAAAAA3M/Zm10eyqlAtQ/s72-c/plurals.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-585926241537941570
blogger_orig_url: http://www.zdominguez.com/2010/09/string-pluralization.html
---

Last week, I discovered Android's support for plural strings by accident. And a good accident it was since I am working on an app that will display a float to the user. I used to display:

```java
You set XX mile(s).
```

which is kinda lame.

[Plurals](http://developer.android.com/guide/topics/resources/string-resource.html#Plurals) lets you specify the string to display for different quantities. So how do we use this Plurals thing?

In your string resources XML, which is usually `strings.xml`, define the `plurals` element like so:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
<plurals name="pluralsTest">
   <item quantity="one">You have one friend.</item>
   <item quantity="other">You have %d friends.</item>
</plurals>
</resources>
```

Remember, your parent node must be `<resources>`!

Android provides several methods to use these in your code. Let's see what each of them results to when used:

```java
// set one as the quantity
String one = getResources().getQuantityString(R.plurals.pluralsTest, 1);

// set two as the quantity
String more = getResources().getQuantityString(R.plurals.pluralsTest, 2, 2);

// set one as the quantity
CharSequence quantity = getResources().getQuantityText(R.plurals.pluralsTest, 1);

// set two as the quantity
CharSequence quantityMore = getResources().getQuantityText(R.plurals.pluralsTest, 2);
```

I created a basic layout with `TextView`s to display what each of these strings look like. Take note though that `getQuantityText()` returns a `CharSequence` and not a `String`! Also, from what I have noticed, and as the name mildly suggests, `getQuantityText()` gets the actual value of the text you defined in your xml.

![](http://2.bp.blogspot.com/_p3DeelS9Gp8/TJDhPBxbZ2I/AAAAAAAAA3M/Zm10eyqlAtQ/s320/plurals.png)![](http://2.bp.blogspot.com/_p3DeelS9Gp8/TJDhPBxbZ2I/AAAAAAAAA3M/Zm10eyqlAtQ/s1600/plurals.png)
