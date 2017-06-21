---
layout: post
title: "Children, Respect Your Parent(s)"
tags:
- android
- quick tip
---
I was updating a bit of code the other day that involved dynamically inflating views into a `LinearLayout` using `DataBindingUtil.inflate(LayoutInflater.from(context), R.layout.row_related_property, container, false)`.

I decided to jump in and update it to use the generated binding's inflate method: `RowRelatedProperty.inflate(LayoutInflater.from(context))` instead.

This inflated layout has margins set on it's root, similar to this:
```xml
<LinearLayout
        android:orientation="vertical"
        android:layout_marginTop="16dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <!-- more views here -->
</LinearLayout>
```

Happy with my changes, I ran the app and immediately noticed a problem. All the views are smooshed together! Show layout bounds tell me that there is no margin being set at all.

My first thought was that generated binding inflation behaves differently. Odd. I looked through the code and it seems to call through to the same method. Weird.

With a little help from my friends (holla [Hugo!](https://twitter.com/botteaap)), I was quickly pointed out the error of my ways. It is important to respect your parent(s)! And by respect I mean pass it on into `inflate`.

One thing I always forget is that any `layout_*` attribute is an instruction to the **parent**. Whenever I am reminded of this, I always go "Ah yes _of course_ I knew _that_".

<p style="text-align: center"><a href="inflate params"><img src="{{ site.baseurl }}/assets/inflate_layout_params.png" height="360" ></a></p>

The code that generated the screenshot above is in my [Sandbox repo](https://github.com/zmdominguez/sdk_sandbox/pull/9/commits/b2d248fc81e56fa9bb81c0f19203698f3b0360ca).

If you want to read more about views, layouts, and attributes, go ahead and read Ian Lake's [excellent Medium post](https://medium.com/google-developers/layouts-attributes-and-you-9e5a4b4fe32c) on the topic.