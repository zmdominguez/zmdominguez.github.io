---
layout: post
title: 'Quick Tip: Understanding Alternate Resources'
date: '2014-06-08T04:38:00.002+10:00'
author: Zarah Dominguez
tags:
  - resources
  - quick tips
  - android
  - layout
modified_time: '2014-06-10T19:52:39.583+10:00'
thumbnail: http://4.bp.blogspot.com/-XNEAgcrexII/U5NYOKwYXaI/AAAAAAAABKU/BEDHGcrgxJc/s72-c/Screen+Shot+2014-06-08+at+2.21.07+AM.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-8282460016159553910
blogger_orig_url: http://www.zdominguez.com/2014/06/quick-tip-understanding-alternate.html
---

Trying to support as many devices as possible the best way possible is a very daunting task indeed. You will usually need to provide a lot of different layouts, strings, or dimensions (among others) to make your app look great whatever the user's device is. And then you start chaining resource qualifiers and testing which resource is being loaded by the OS can become a nightmare very quickly.

Here's a trick I started using which seemed to work quite well. Create a string (the app name works well if you have an Action Bar) that you can display on-screen (or just throw into a `Log` or a `Toast`) that will quickly let you know from which of those very many `/res` folders your resources are being pulled out of.

For this demo, I have the following `/res/values-xxxxx` folders:
<div class="separator" style="clear: both; text-align: center;"><a href="http://4.bp.blogspot.com/-XNEAgcrexII/U5NYOKwYXaI/AAAAAAAABKU/BEDHGcrgxJc/s1600/Screen+Shot+2014-06-08+at+2.21.07+AM.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://4.bp.blogspot.com/-XNEAgcrexII/U5NYOKwYXaI/AAAAAAAABKU/BEDHGcrgxJc/s1600/Screen+Shot+2014-06-08+at+2.21.07+AM.png" /></a></div>

As you can see, there are `strings.xml` files in each of the configurations I want to support. Each of these files will contain whatever description we want to see on the device or in the `Log`s. In my case, I have custom strings for each form factor and orientation, which allows me to validate things easier. But this is particularly useful if the changes per form factor and orientation is subtle, such as dimensions.

Here's what it looks like for phones:

| | |
| -- | -- |
| <a href="http://4.bp.blogspot.com/-UEeCvpIO91I/U5NWVNm8BjI/AAAAAAAABJs/N9DhjUBNyp8/s1600/device-2014-06-08-015715.png" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"><img border="0" src="http://4.bp.blogspot.com/-UEeCvpIO91I/U5NWVNm8BjI/AAAAAAAABJs/N9DhjUBNyp8/s320/device-2014-06-08-015715.png" /></a> | <a href="http://2.bp.blogspot.com/-ox7KE0fQL1k/U5NWVRZtihI/AAAAAAAABJ0/TUXOTojC9-k/s1600/device-2014-06-08-015755_phone_landscape.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://2.bp.blogspot.com/-ox7KE0fQL1k/U5NWVRZtihI/AAAAAAAABJ0/TUXOTojC9-k/s320/device-2014-06-08-015755_phone_landscape.png" /></a> |


And here are the outputs for tablets:

| | |
| --- | --- |
| <a href="http://2.bp.blogspot.com/-68wEt8q3YmE/U5NWWP901rI/AAAAAAAABJ8/-yXodg79Ld8/s1600/device-2014-06-08-020006_tablet_portrait.png" imageanchor="1"><img border="0" src="http://2.bp.blogspot.com/-68wEt8q3YmE/U5NWWP901rI/AAAAAAAABJ8/-yXodg79Ld8/s320/device-2014-06-08-020006_tablet_portrait.png" /></a> | <a href="http://2.bp.blogspot.com/-FgfXNNCOl_w/U5NWVYwxPDI/AAAAAAAABJw/ILNSixSpC-c/s1600/device-2014-06-08-015946_tablet_landscape.png" imageanchor="1"><img border="0" src="http://2.bp.blogspot.com/-FgfXNNCOl_w/U5NWVYwxPDI/AAAAAAAABJw/ILNSixSpC-c/s320/device-2014-06-08-015946_tablet_landscape.png" /></a> |

Each of the `strings.xml` files contains just these two strings (sample taken from `/res/values-sw800dp`):
```xml
<resources>
    <string name="app_name">MultiDeviceSupport - Tablet Portrait</string>
    <string name="hello_world">Hello world on a tablet!</string>
</resources> 
```

Have fun debugging!
