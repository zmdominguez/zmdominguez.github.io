---
layout: post
title: Adding a float value to your resources
date: '2012-04-13T16:49:00.001+10:00'
author: Zarah Dominguez
tags:
- float
- R.java
- quick tips
- android
modified_time: '2012-04-13T16:49:38.982+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-1473979132439961563
blogger_orig_url: http://www.zdominguez.com/2012/04/adding-float-value-to-your-resources.html
---

Earlier today, I was trying to figure out how to add a float value to constants file. What I used to do was add a string value in my `strings.xml`, retrieve the string value, and convert it to float.
```java
float floatFromFile = Float.valueOf(getResources().getString(R.string.my_float));
```

I was trying out something new but it wasn't working, so I decided to look for a more accepted solution. Google led me to this [StackOverflow question](http://stackoverflow.com/questions/3282390/add-floating-point-value-to-android-resources-values). I was on the right track after all! I think the accepted answer is incomplete, or not clear enough for my purposes.

I ended up having this entry in my `dimensions.xml` file:
```xml
<item name="face_feature_marker_size" type="vals" format="float">2.0</item>
```

And then in my code, I retrieve the value as:
```java
TypedValue tempVal = new TypedValue();
getResources().getValue(R.vals.face_feature_marker_size, tempVal, true);
float markerSize = testVal.getFloat();</pre>
```

I ended up having more lines of code with no idea if this is more optimized. Let me know what you think!