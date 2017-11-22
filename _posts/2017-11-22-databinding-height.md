---
layout: post
title: "Binding to the H(eight)"
tags:
- android
- data binding
---
As would probably be obvious by now, I have been investing a lot of time in learning and using data binding.

Yesterday, we were trying to bind a view's height and width to a value from a model. Directly using data binding did not work, and Googling for the issue says we have to do two things:
- Provide a [default value](https://stackoverflow.com/a/35332242/395576)
- [Create a `BindingAdapter`](https://issuetracker.google.com/issues/37054474)

Remember that data binding strips out any data binding-related things in the XML. Without a `default`, the widget is left without a height and width. Hence the importance of including a `default` value.

```xml
<ImageView
    android:id="@+id/agent_photo"
    android:layout_width="@{entry.agentPhotoSize, default=@dimen/default_photo_size}"
    android:layout_height="@{entry.agentPhotoSize, default=@dimen/default_photo_size}" />
```

We set about creating the `BindingAdapter` as illustrated in the issue tracker. But running the code shows us that the height being set to the view is a massive value. The view seems to go on and on forever. Not good.

I was ready to give up and chalk it up to "Welp maybe `RecyclerView` does not like what we're doing." But I can't just give up like that! But then suddenly... :bulb:

I don't know how I missed it the first time, but I realised that we are passing in a dimension. Our `BindingAdapter` _does_ work. But it is using the raw value of the dimension resource as the view's height. Duh. Resolving the resource to the actual value makes us all happy again.

```java
@BindingAdapter({"android:layout_height"})
public static void setAgencyBrandingHeight(View view, @DimenRes int height) {
    ViewGroup.LayoutParams layoutParams = view.getLayoutParams();
    layoutParams.height = view.getContext().getResources().getDimensionPixelSize(height);
    view.setLayoutParams(layoutParams);
}
```

And then of course after figuring this out, I remembered that I already wrote about [using resources]({{ site.baseurl }}{% link _posts/2016-07-19-using-resource-ids-in-data-binding.md %}). :no_good: