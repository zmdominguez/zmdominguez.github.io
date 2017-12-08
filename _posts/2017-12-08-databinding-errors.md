---
layout: post
title: "Parsing Data Binding Errors"
tags:
- android
- data binding
---
Learning something new is always fun and exciting. That is, until seemingly cryptic error messages start creeping up.

For the past year, I have been helping my teammates get more acquainted with data binding. One topic that frequently comes up is that if something goes wrong, it can be hard to figure out exactly _why_.

I have talked a lot about data binding, and the past couple of times that I did, I decided to include a section on a few of the errors I see most often.

<center><h3>1</h3></center>

<p style="text-align: center"><a href="{{ site.baseurl }}/assets/databinding_errors/user_defined_types.png"><img src="{{ site.baseurl }}/assets/databinding_errors/user_defined_types.png" ></a></p>

This error means the layout file is using a variable ("`user defined types`") called `library`, but it is has not been declared within the `data` element.

To fix this, make sure that any variables being used in the layout file have been declared. If they are, double check that there are no typos as well!

```xml
<data>
    <variable
        name="library"
        type="io.plaidapp.ui.AboutActivity.Library" />
</data>
<io.plaidapp.ui.widget.BaselineGridTextView
   android:id="@+id/library_description"
   android:text="@{library.description}" />
```

It turns out that I have reported this in the [issue tracker](https://issuetracker.google.com/issues/62685775), hopefully the team gets around to it soon! :smile:

<center><h3>2</h3></center>

<p style="text-align: center"><a href="{{ site.baseurl }}/assets/databinding_errors/namespace_ignore.png"><img src="{{ site.baseurl }}/assets/databinding_errors/namespace_ignore.png" ></a></p>

If I remember correctly, during the early days of data binding, the `BindingAdapter` implementations would have the namespace included in the annotations. It did seem to ignore it, and at some point in the past year, this warning popped up.

It is not an error, but if you want to fix the warning, remove the namespace declaration and everything should be good to go!

```java
@BindingAdapter({"imageUrl", "circleCrop"})
public static void setAvatar(ImageView imageView, String url, boolean isCircleCropped) {
   // Do stuff
}
```

<center><h3>3</h3></center>

<p style="text-align: center"><a href="{{ site.baseurl }}/assets/databinding_errors/observable_fields.png"><img src="{{ site.baseurl }}/assets/databinding_errors/observable_fields.png" ></a></p>

`Observable` fields may be one of my most favourite things in data binding. If you are not familiar, I highly suggest [reading up on them](https://developer.android.com/topic/libraries/data-binding/index.html#observablefields) and trying them out.

To fix this warning, check that any `Observable` being used in a `BindingAdapter` uses the values ("`contents`") directly.

As an example, say I have an `ObservableBoolean` called `shouldLoadUserAvatar` that I use in a layout file like this:
```xml
app:shouldLoadUserAvatar="@{dribbbleState.shouldLoadUserAvatar}"
```

In the `BindingAdapter` implementation, you can (and should) use a `Boolean` to get the value that we want:
```java
@BindingAdapter({"userAvatar", "shouldLoadUserAvatar"})
public static void setPlayerCommentAvatar(ImageView imageView, String userAvatar, Boolean shouldLoadUserAvatar) {
   if (shouldLoadUserAvatar) {
       // Do stuff
   }
}
```

If there are more errors you see whilst using data binding, please share them (and how to fix them, if you can!) in the comments below!