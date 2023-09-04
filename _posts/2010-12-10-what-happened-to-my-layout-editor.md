---
layout: post
title: What happened to my layout editor?
date: '2010-12-10T14:13:00.003+11:00'
author: Zarah Dominguez
tags:
  - eclipse
  - XML
  - android
  - layout
modified_time: '2010-12-10T14:21:32.084+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-1989098572060089139
blogger_orig_url: http://www.zdominguez.com/2010/12/what-happened-to-my-layout-editor.html
---

There you are, happily creating your layout files in the Eclipse plug-in's layout editor. Dragging and dropping is a breeze.  But then one day, you open a layout XML file and boom! No UI!  All you see is the XML tree with all the nodes and attributes.  What happened?

This happened to me and I was in a panic for a few seconds.  Why do things like these have to happen to me?  I tried opening a layout file from another project in the same workspace, and it has the UI! What happened?

It probably has something to do with the interpreter you used to open the XML file, a voice in my head said. So I tried right-clicking on the XML file, and lo and behold, I found it. I may have accidentally clicked on some file in one of my mad-clicking moments and changed the setting.

So anyway, to bring back the Layout UI Editor, right click on an XML file > Choose Open With > Android Layout Editor.

I would say that everything is handy dandy, but apparently, the engineers at Google decided that we developers need a little less help and removed the **very useful** up and down arrow keys in the Outline View when editing XML layouts. Why do they hurt us like this?

I WANT MY UP/DOWN KEYS BACK!