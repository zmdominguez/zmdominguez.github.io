---
layout: post
title: Using resource IDs in data binding layouts
date: '2016-07-19T14:11:00.004+10:00'
author: Zarah Dominguez
tags:
- data binding
- resources
- android
modified_time: '2016-07-19T15:00:34.823+10:00'
thumbnail: https://1.bp.blogspot.com/-K7hsk5AlZOo/V42jxWTtoqI/AAAAAAAAeWI/x4BEoqoeVSscOidyZd9NlwHobVhGlHcxQCLcB/s72-c/device-2016-07-17-225953.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-316193729330520012
blogger_orig_url: http://www.zdominguez.com/2016/07/using-resource-ids-in-data-binding.html
---

I have been playing with data binding more and more over the last couple of weeks. This week, it's all about creating a dialog with stuff dictated by a value from an enum.

Each value in [this enum](https://github.com/zmdominguez/sdk_sandbox/blob/master/app/src/main/java/com/zdominguez/sdksandbox/models/AdventureTimeCharacters.java) has the following properties:

```java
@IdRes int textView
@StringRes int quote
@StringRes int name
@ColorRes int colour
```

I have added a new button in my [Sandbox app](https://github.com/zmdominguez/sdk_sandbox) to choose one value from the enum and display the properties in an alert dialog via data binding. Ambitious, I know!

The relevant Java part is mainly just creating and showing the dialog:

```java
@OnClick(R.id.data_binding_alert)
public void onSendDataBoundAlert() {
   AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this, R.style.MaterialAlertDialog);
   builder.setPositiveButton(android.R.string.ok, null)
           .setNegativeButton("Not now", null);
   View view = LayoutInflater.from(MainActivity.this).inflate(R.layout.dialog_data_binding_demo, null);
   DialogDataBindingDemoBinding binding = DataBindingUtil.bind(view);
   binding.setCharacter(getRandomCharacter());
   builder.setView(view);

   builder.create().show();
}
```

The tricky part is telling data binding how to use our resource IDs. Say I want the colour to be set as a background for a `TextView`, and the name to be displayed:

```xml
<TextView
     android:id="@+id/name"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:minHeight="64dp"
     android:textSize="24sp"
     android:textStyle="bold"
     android:gravity="center_vertical"
     android:paddingLeft="16dp"
     android:background="@{character.colour}"
     android:text="@{character.name}"/>
```

Running the sample though, you'll see that the colour is not set and a dark grey-ish lilac-ish colour is used instead:

<p><img src="https://1.bp.blogspot.com/-K7hsk5AlZOo/V42jxWTtoqI/AAAAAAAAeWI/x4BEoqoeVSscOidyZd9NlwHobVhGlHcxQCLcB/s640/device-2016-07-17-225953.png"></p>

Unacceptable indeed!

So how do we make the Earl of Lemongrab happy? There are two ways of solving this problem.

**A. Use a BindingAdapter**

A [`BindingAdapter`](https://developer.android.com/reference/android/databinding/BindingAdapter.html) lets you manipulate and do a bit more involved logic on your data before applying it to the `View`. To use a `BindingAdapter`, first create a static method in your code that is bound to either a standard Android attribute or a custom one.

I create a custom attribute here called `characterBackground`:

```java
@BindingAdapter({"characterBackground"})
public static void characterBackground(TextView textView, AdventureTimeCharacters character) {
     textView.setBackgroundColor(ContextCompat.getColor(textView.getContext(), character.getColour()));
}
```

You can then use this `BindingAdapter` in the `TextView`:

```xml
app:characterBackground="@{character}"
```

Do not forget to add the app namespace! Android Studio can add this for you. Just type in `appNs` and it will autocomplete.

This solution works, but is a bit too involved. And you said data binding is easy!!!

Well it kinda is, because, it turns out that we can:
**B. Set the value in XML**

But it didn't work! Well it turns out the problem is because we are passing a resource ID into an attribute that expects a resolved color value. So the only thing we need to do is to move all that code in our `BindingAdapter` into XML.

First add the import:

```xml
<data>
    . . .
    <import type="android.support.v4.content.ContextCompat" />
</data>
```

And then use it:

```xml
<TextView
     android:id="@+id/name"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:minHeight="64dp"
     android:textSize="24sp"
     android:textStyle="bold"
     android:gravity="center_vertical"
     android:paddingLeft="16dp"
     android:background="@{ContextCompat.getColor(context, character.colour)}"
     android:text="@{character.name}"/> 
```

Very convenient! And it works! Note that you do not have to import anything to use `context`, there is some data binding magic going on that automatically injects it for you if you call it. Think of it as The Beckoning of the Context.

<p><img src="https://1.bp.blogspot.com/-00MWkXjIQcA/V42oBcznX-I/AAAAAAAAeWY/WmXbW-AmRdsEwr_v-g9v0FwuZ5LgHnmFwCLcB/s640/device-2016-07-19-140855.png"></p>

The Earl is now happy.

PS: If you need to set a drawable from a resource ID as an `ImageView` src, I found that you would also need to do this trick, aka:

```xml
android:src="@{ContextCompat.getDrawable(context, character.icon)}"
```