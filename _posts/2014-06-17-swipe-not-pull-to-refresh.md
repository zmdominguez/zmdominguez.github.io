---
layout: post
title: Swipe, not Pull, to Refresh
date: '2014-06-17T18:29:00.000+10:00'
author: Zarah Dominguez
tags:
  - listview
  - swipe to refresh
  - android
modified_time: '2014-06-17T18:31:44.827+10:00'
thumbnail: http://2.bp.blogspot.com/-d7fqvs0jkfE/U5_4iC4wsGI/AAAAAAAABK0/Tli1cee-h_I/s72-c/device-2014-06-17-160825.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-7773016512477593979
blogger_orig_url: http://www.zdominguez.com/2014/06/swipe-not-pull-to-refresh.html
---

I have recently came across this new [View in the support library package](https://developer.android.com/reference/android/support/v4/widget/SwipeRefreshLayout.html) that allows your app to have built-in support for <strike>pull</strike> swipe to refresh. This is pretty cool, since we don't have to use any of the libraries out there. Admittedly, very little customization can be done, but then what else can we customize, right?

Anyway, here's a short demo of using this nifty little view.

| Initial list                                                                                                                                                                                                                                                                                                                                             | While refreshing                                                                                                                                                                                                                                                                                                                                         | After refreshing                                                                                                                                                                                                                                                                                                                                         |
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <a href="http://2.bp.blogspot.com/-d7fqvs0jkfE/U5_4iC4wsGI/AAAAAAAABK0/Tli1cee-h_I/s1600/device-2014-06-17-160825.png" imageanchor="1" style="margin-left: auto; margin-right: auto;"><img border="0" src="http://2.bp.blogspot.com/-d7fqvs0jkfE/U5_4iC4wsGI/AAAAAAAABK0/Tli1cee-h_I/s1600/device-2014-06-17-160825.png" height="320" width="180" /></a> | <a href="http://1.bp.blogspot.com/-86wrbovsdCg/U5_4iN-UqrI/AAAAAAAABKs/5paup4tIsDY/s1600/device-2014-06-17-160903.png" imageanchor="1" style="margin-left: auto; margin-right: auto;"><img border="0" src="http://1.bp.blogspot.com/-86wrbovsdCg/U5_4iN-UqrI/AAAAAAAABKs/5paup4tIsDY/s1600/device-2014-06-17-160903.png" height="320" width="180" /></a> | <a href="http://1.bp.blogspot.com/-_V06YWPycfY/U5_4iHsE-0I/AAAAAAAABKw/Q48SiFs2HJQ/s1600/device-2014-06-17-160918.png" imageanchor="1" style="margin-left: auto; margin-right: auto;"><img border="0" src="http://1.bp.blogspot.com/-_V06YWPycfY/U5_4iHsE-0I/AAAAAAAABKw/Q48SiFs2HJQ/s1600/device-2014-06-17-160918.png" height="320" width="180" /></a> |

The app is a simple ListView that shows a list of countries. Swiping down on the list will simulate a long-running activity (like connecting to a server, for example) and afterwards updating the list with new data.

Adding support for swipe to refresh is pretty straightforward. You would have to wrap the swipe-able layout in a `SwipeRefreshLayout`. Here is my XML file:
```xml
<android.support.v4.widget.SwipeRefreshLayout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools" />
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="MergeRootFrame">
        <ListView android:id="@android:id/list"
                  android:layout_width="match_parent"
                  android:layout_height="match_parent" />
</android.support.v4.widget.SwipeRefreshLayout>
```

In your `Fragment`, you would then have to put an `onRefreshListener` on this `SwipeRefreshLayout`. In my `Fragment`'s `onCreateView`, I have this:
```java
// Configure the swipe refresh layout
mSwipeRefreshLayout = (SwipeRefreshLayout) rootView.findViewById(R.id.container);
mSwipeRefreshLayout.setOnRefreshListener(this);
mSwipeRefreshLayout.setColorScheme(
        R.color.swipe_color_1, R.color.swipe_color_2,
        R.color.swipe_color_3, R.color.swipe_color_4);
```

The colors are defined in a `colors.xml` file in my `res/values` folder. To show or hide the refresh animation, you would have to call `setRefreshing(boolean)`. This means that if you have to kick off an `AsyncTask` when the user swipes down, call `setRefreshing(true)` in `onPreExecute`, and call `setRefreshing(false)` in `onPostExecute`.

The implementation of `onRefresh` in the demo app is pretty simple, it simply grabs the next bunch of countries from a pre-defined list.

```java
@Override
public void onRefresh() {
    // Start showing the refresh animation
    mSwipeRefreshLayout.setRefreshing(true);
    
    // Simulate a long running activity
    new Handler().postDelayed(new Runnable() {
        @Override
        public void run() {
            updateCountries();
        }
    }, 5000);
}
```

That's it really. Nice and short. The code for this demo is [in Github](https://github.com/zmdominguez/swipe-to-refresh-demo).