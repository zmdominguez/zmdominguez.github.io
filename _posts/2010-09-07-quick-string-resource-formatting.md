---
layout: post
title: Quick string resource formatting
date: '2010-09-07T00:44:00.000+10:00'
author: Zarah Dominguez
tags:
- formatter
- textview
- android
modified_time: '2010-09-07T01:23:39.440+10:00'
thumbnail: http://4.bp.blogspot.com/_p3DeelS9Gp8/TIUCLKqQbGI/AAAAAAAAA1Y/6YbcbebVGeg/s72-c/string_formatter.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-2286823627152758093
blogger_orig_url: http://www.zdominguez.com/2010/09/quick-string-resource-formatting.html
---

Sooner or later, you would want to display a message to your user with dynamic content. This may be the number of results, the user's name, etc.

Luckily for us, Android provides a convenience method that we can use for such purposes.

```java
public final String getString (int resId, Object... formatArgs)
```

This means that we can define a string in our `strings.xml` file with format specifiers supported by [Java's formatter class](http://developer.android.com/reference/java/util/Formatter.html). For example, if I have such a string:</div>

```xml
<string name="formatted_string">Hello, %s! You have %d messages.</string>
```

I can get this string, apply the formatting, and then set it into a TextView without additional processing on my part.


```java
TextView string = (TextView) findViewById(R.id.form_string);
string.setText(getString(R.string.formatted_string, "Zarah", 4));
```

And I will have this:

[![](http://4.bp.blogspot.com/_p3DeelS9Gp8/TIUCLKqQbGI/AAAAAAAAA1Y/6YbcbebVGeg/s320/string_formatter.png)](http://4.bp.blogspot.com/_p3DeelS9Gp8/TIUCLKqQbGI/AAAAAAAAA1Y/6YbcbebVGeg/s1600/string_formatter.png)Nifty and easy!

Of course, this is a simple example. But I hope you get the drift. Do read the Formatter's documentation to see all possible formats you can use.
