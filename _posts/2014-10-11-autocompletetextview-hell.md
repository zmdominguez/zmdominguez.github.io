---
layout: post
title: AutoCompleteTextView Hell
date: '2014-10-11T03:08:00.000+11:00'
author: Zarah Dominguez
tags:
  - EditText
  - android
modified_time: '2014-10-11T03:26:19.561+11:00'
thumbnail: http://2.bp.blogspot.com/-PQMuNe2ei20/VDf_p-JUWiI/AAAAAAAAAE4/49UqS7AS6dA/s72-c/device-2014-10-11-024214.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-4988538796002882744
blogger_orig_url: http://www.zdominguez.com/2014/10/autocompletetextview-hell.html
---

Today, I ran into a weird "feature" of Android. I was working on an [AutoCompleteTextView](http://developer.android.com/reference/android/widget/AutoCompleteTextView.html) with the dropdown list having section dividers. It all works well in portrait mode, but gets all messed up in landscape.

I made a sample app to illustrate the point of this blog [[Github repo](https://github.com/zmdominguez/autocompletetextviewhell)]. Clone it, run it, rotate the phone, select an item from the suggested auto-complete results, and get your mind blown. Or your heart stabbed. Or your stomach sucker-punched. Your choice.

So what was happening? Here's what.

When we tell the app to perform a filter, we construct an array of the resulting matches. The example is pretty straightforward, just look for countries that start with whatever the user has typed in.

```java
// Filter by start of string
String country = mCountries.get(i);
if(country.toLowerCase().startsWith(constraint.toString().toLowerCase())) {
    mFilteredData.add(country);
}
```

You can make this filtering as complicated as you like or need, just make sure to pass in whatever the user needs to see.

To illustrate having section headers (aka disabled items), we insert dummy text every fifth place in the list. The sample app does not care if there are more results after a section header, we still insert anyway.

So. Let's filter. Typing in "pa" will give us this set of data:

```shell
10-11 02:45:46.028  16282-20011/com.blogspot.droidista.autocompletetextviewhell D/AutoCompleteFragment﹕
Filtered results: [Section!, Pakistan, Palestine, Panama, Papua New Guinea, Section!, Paraguay]
```

There are two sections and five countries. Remember this.

| Position in list | Value |
| --- | --- |
| 0 | Section! |
| 1 | Pakistan |
| 2 | Palestine |
| 3 | Panama |
| 4 | Papua New Guinea
| 5 | Section!
| 6 | Paraguay

This screenshot shows how the results are rendered in portrait mode. So far, so good. Selecting an item from the list populates the EditText with the country's name.
<div class="separator" style="clear: both; text-align: center;"><a href="http://2.bp.blogspot.com/-PQMuNe2ei20/VDf_p-JUWiI/AAAAAAAAAE4/49UqS7AS6dA/s1600/device-2014-10-11-024214.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://2.bp.blogspot.com/-PQMuNe2ei20/VDf_p-JUWiI/AAAAAAAAAE4/49UqS7AS6dA/s1600/device-2014-10-11-024214.png" height="320" width="180" /></a></div>

Now let's try rotating our phone. This is where things get juicy. First off, there are no section headers. Second, the results are not in alphabetical order anymore. If we debug all over the adapter, we see what is written on the screenshot: Item on the left is position = 1, item in the middle is position = 0, and item on the right is position = 2.

Remember the result set we have? "Pakistan" is definitely NOT in position = 0, it should have been a section header! If we go ahead and select "Pakistan" in the suggestions above the keyboard, the EditText will populate with the item in position = 0 of the result set, i.e. "Section!". Definitely not good.

<div class="separator" style="clear: both; text-align: center;"><a href="http://2.bp.blogspot.com/-Cae8yAlM1jc/VDf_qQXMPBI/AAAAAAAAAE8/DP1Y12iS7XU/s1600/device-2014-10-11-024313_land.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://2.bp.blogspot.com/-Cae8yAlM1jc/VDf_qQXMPBI/AAAAAAAAAE8/DP1Y12iS7XU/s1600/device-2014-10-11-024313_land.png" height="180" width="320" /></a></div>

The AutoCompleteTextView widget has been around since API level 1, but I haven't messed around with it as much as I did today. Googling returns very, very sparse results on this topic.
- Does this mean people do not have the same problem as I did?
- No one uses section headers in AutoCompleteTextViews?
- No one uses this screen mode (EditText in full screen) in landscape without setting IME flags?
- Is there a secret trick to making this work out-of-the-box?
- Or the most plausible of all,
- Am I being stupid?