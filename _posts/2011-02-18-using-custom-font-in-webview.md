---
layout: post
title: Using a custom font in WebView
date: '2011-02-18T13:49:00.009+11:00'
author: Zarah Dominguez
tags:
  - css
  - formatter
  - font
  - webview
  - android
modified_time: '2011-05-28T04:11:22.776+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-2462635597454741075
blogger_orig_url: http://www.zdominguez.com/2011/02/using-custom-font-in-webview.html
---

In one of my projects, I needed to display some special characters that the Android OS by itself cannot seem to render. I figured that I would need to provide a custom font that includes the characters that I needed.

If I was using a `TextView`, I could use [TextView#setTypeFace](http://developer.android.com/reference/android/widget/TextView.html#setTypeface(android.graphics.Typeface)).  But I was using a `WebView` and I feared that things would be more complicated than that.  So how do I do this?

Here's how we can make it work.

**Step 1**: We would need to have our font face included in our project's `/assets` folder. So look for a TTF file that you can use freely, make sure the author/creator allows you to re-distribute it!

**Step 2**: Edit your HTML file to include some CSS stuff, just so the `WebView` would know what font you want to use.  Here's a sample file:
```xml
<html>
    <head>
        <link href="YourCssFile.css" rel="stylesheet" type="text/css"/>
    </head>
    <body>
        <span class="phon">This string contains special characters: əˈpåstrəfi </span>
    </body>
</html>
```
Make sure that the `href` references are correct.  In this case, my CSS file, HTML file and font file are in the same folder.

**Step 3**: Define your CSS file. In this case, our `YourCssFile.css` would be:
```css
@font-face {
    font-family: "My font";
    src: url('MyFontFile.TTF');
}

.phon, .unicode {
    display: inline;
    font-family: 'My font', Verdana, sans-serif;
    font-size: 14pt;
    font-weight: 500;
    font-style: normal;
    color: black;
}
```

**Step 4**: Load the file in your `WebView`! You can use
```java
WebView webView = (WebView) findViewById(R.id.myWebview);
webView.loadUrl("path/to/html/file");
```
or
```java
webView.loadDataWithBaseURL("file:///android_asset/",
article, "text/html", Encoding.UTF_8.toString(), null);
```

If you will use the second option, your `article` variable would contain the HTML string. Just make sure that you escape your quotation marks with a backslash (\).

**IMPORTANT NOTE:** This feature seems to be broken in Android 2.0 and 2.1, as reported [here](http://code.google.com/p/android/issues/detail?id=4448).
{: .notice--primary}