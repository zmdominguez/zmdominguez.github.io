---
layout: post
title: Missing hierarchyviewer in SDK 7
date: '2010-10-26T19:44:00.007+11:00'
author: Zarah Dominguez
tags:
  - wtfSDK
  - android
  - hierarchyviewer
modified_time: '2010-11-10T17:27:47.216+11:00'
thumbnail: http://1.bp.blogspot.com/_p3DeelS9Gp8/TMaV5ddXBTI/AAAAAAAAA3U/m64RuYWMdOI/s72-c/SDK+version.PNG
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-928865941430702707
blogger_orig_url: http://www.zdominguez.com/2010/10/missing-hierarchyviewer-in-sdk-7.html
---

If you have SDK version 7, you are most probably missing the <code>hierarchyviewer</code> from your <code>/tools</code> folder.  To check your SDK version, launch the SDK manager UI from your installation path, usually <code>C:\android-sdk-windows</code>, then click About.<a onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}" href="http://1.bp.blogspot.com/_p3DeelS9Gp8/TMaV5ddXBTI/AAAAAAAAA3U/m64RuYWMdOI/s1600/SDK+version.PNG"><img style="display:block; margin:0px auto 10px; text-align:center;cursor:pointer; cursor:hand;width: 320px; height: 93px;" src="http://1.bp.blogspot.com/_p3DeelS9Gp8/TMaV5ddXBTI/AAAAAAAAA3U/m64RuYWMdOI/s320/SDK+version.PNG" border="0" alt="" id="BLOGGER_PHOTO_ID_5532274006664086834" /></a>To run <code>hierarchyviewer</code>, you need to manually create the </code>hierarchyviewer.bat</code> file and add it to your <code>&lt;install_path&gt;/tools</code> directory.  The text of the batch file can be <a href="http://android.git.kernel.org/?p=platform/sdk.git;a=blob_plain;f=hierarchyviewer/etc/hierarchyviewer.bat;hb=tools_r7">copied from here</a>.And so, you can now run <code>hierarchyviewer</code> as you would if the SDK release isn't effed up.  Don't know how to run it? Follow these steps:a. In Windows, open up a terminal by running <code>cmd</code>.b. Navigate to your SDK's install path.  Since I installed mine in <code>C:\</code>, I would have to type in <code>cd C:\android-sdk-windows\tools</code>c. Type in <code>hierarchyviewer</code> at the prompt.