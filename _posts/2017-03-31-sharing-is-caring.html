---
layout: post
title: Sharing is Caring
date: '2017-03-31T21:08:00.002+11:00'
author: Zarah Dominguez
tags:
- intents
- share
- android
modified_time: '2017-03-31T21:28:58.617+11:00'
thumbnail: https://2.bp.blogspot.com/-gktN948vHGc/WN4kQwfKPSI/AAAAAAABBcU/HWyPcR5RSVM81c3BVW7yVamvnJkMbdB7QCLcB/s72-c/device-2017-03-31-203350.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-9168071053130527543
blogger_orig_url: http://www.zdominguez.com/2017/03/sharing-is-caring.html
---

Or so they say.

Luckily for us, Android makes it [super easy to share things](https://developer.android.com/training/sharing/send.html) between apps. It even provides a pretty-looking Intent chooser*!

[![](https://2.bp.blogspot.com/-gktN948vHGc/WN4kQwfKPSI/AAAAAAABBcU/HWyPcR5RSVM81c3BVW7yVamvnJkMbdB7QCLcB/s320/device-2017-03-31-203350.png)](https://2.bp.blogspot.com/-gktN948vHGc/WN4kQwfKPSI/AAAAAAABBcU/HWyPcR5RSVM81c3BVW7yVamvnJkMbdB7QCLcB/s1600/device-2017-03-31-203350.png)

We can get this nifty bottom sheet with only a few lines of code:

```java
// Construct the intent we want to send
final Intent shareIntent = new Intent(Intent.ACTION_SEND);
shareIntent.setAction(Intent.ACTION_SEND);
shareIntent.putExtra(Intent.EXTRA_TEXT, "This is my text to send.");
shareIntent.setType("text/plain");

// Ask Android to create the chooser for us
final Intent chooser = Intent.createChooser(shareIntent, getString(R.string.share_text));
startActivity(chooser);
```

That's it, your work is done.

But what if we want the users to choose our own app? For example, we have a Share via Domain activity in our app that we prefer our users to use when sharing properties. It generates an email you can send to your friends and includes some information about the property. However, we still want to give users the option to share via other channels.

Luckily for us (we Android devs are _really_ lucky, in case you haven't noticed), there is an API that allows us to do just that.

We can add extras (via [`EXTRA_INITIAL_INTENTS`](https://developer.android.com/reference/android/content/Intent.html#EXTRA_INITIAL_INTENTS)) 
to the `chooser Intent` that will tell the OS to prioritise the `Intent`s we want. In my [sandbox app](https://github.com/zmdominguez/sdk_sandbox/pull/7), I made a simple activity that will display the text we send in `shareIntent` above.

```java
Intent customSharer = new Intent(this, CustomShareActivity.class);
Intent[] initialIntents = new Intent[] {customSharer};
chooser.putExtra(Intent.EXTRA_INITIAL_INTENTS, initialIntents);
```

This will surface our priority intents at the very top of the list. It will appear with app icon, with the value we have in `label` from the `Manifest`.

```xml
<activity android:name=".bottomsheet.CustomShareActivity"
          android:label="Choose Me!"/>
```

We end up with something like this:

[![](https://1.bp.blogspot.com/--tVSXIVUS1Y/WN4u5Xvf_rI/AAAAAAABBcw/d_aA31CJf8MwuJhlAZ7v614wbvK3F5o1ACLcB/s320/device-2017-03-31-212608.png)](https://1.bp.blogspot.com/--tVSXIVUS1Y/WN4u5Xvf_rI/AAAAAAABBcw/d_aA31CJf8MwuJhlAZ7v614wbvK3F5o1ACLcB/s1600/device-2017-03-31-212608.png)

Pretty cool, huh?

*I know, the developer docs screenshots are a tad outdated. <span style="background-color: white; color: #545454; font-family: &quot;arial&quot; , sans-serif; font-size: x-small;">¯\_(ツ)_/¯</span>
