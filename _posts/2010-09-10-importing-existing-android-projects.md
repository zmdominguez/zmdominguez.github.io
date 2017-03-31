---
layout: post
title: Importing existing Android projects to Eclipse
date: '2010-09-10T14:34:00.005+10:00'
author: Zarah Dominguez
tags:
- eclipse
- import project
- android
modified_time: '2010-09-10T15:10:13.743+10:00'
thumbnail: http://1.bp.blogspot.com/_p3DeelS9Gp8/TIm8HUy0GDI/AAAAAAAAA1g/Y51qrYJuhOU/s72-c/eclipse-prefs.PNG
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-5466832645122596320
blogger_orig_url: http://www.zdominguez.com/2010/09/importing-existing-android-projects.html
---

When trying to import an existing Android project to Eclipse, I always encounter the error: `The method *XXXXX* must override a superclass method`.

At first I was confused. I think this is a Java 1.5 thing, but I am running Java 1.6, and a quick check with my workspace settings show that this is indeed the case. So why the errors?

I searched the intarwebz, and I found out that I am not the only one encountering this! Now I do not want to comment out all the `@Override` annotations, since it is tedious and would dirty up my code.

Turns out that this is a weird Eclipse behaviour that you can correct with a few simple clicks.

[![](http://1.bp.blogspot.com/_p3DeelS9Gp8/TIm8HUy0GDI/AAAAAAAAA1g/Y51qrYJuhOU/s320/eclipse-prefs.PNG)](http://1.bp.blogspot.com/_p3DeelS9Gp8/TIm8HUy0GDI/AAAAAAAAA1g/Y51qrYJuhOU/s1600/eclipse-prefs.PNG)
Go to Window > Preferences > Java > Compiler. It should show the compiler compliance level to be 1.6, but the errors are still there. What the frak, Eclipse? Just to be sure, select 1.6 in the dropdown options. This worked for me, but if it doesn't work for you, try the next step.

Click the Configure Project Settings link. It should show a list of the projects in your workspace. Choose your Android project, click OK. Tick the Enable project specific settings box, then choose 1.6 as the compiler compliance level. That should work. :)
