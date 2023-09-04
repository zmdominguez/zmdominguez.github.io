---
layout: post
title: Pasting and Extracting Stuff
date: '2015-07-13T01:36:00.001+10:00'
author: Zarah Dominguez
tags:
  - android studio
  - shortcut
modified_time: '2015-07-13T01:36:50.482+10:00'
thumbnail: https://img.youtube.com/vi/bgzPRX9zC30/default.jpg
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-7797253340931010578
blogger_orig_url: http://www.zdominguez.com/2015/07/pasting-and-extracting-stuff.html
---

A lot of times, but especially when I am implementing some new logic, coding for me takes several steps:
1. Write down what I have to do as comments
2. Implement what I have written down
3. Refactor and improve what I have implemented

More often than not, step 3 means moving stuff around, copy-pasting things, extracting variables, defining constants, etc.  I am not a hardcode never-using-my-mouse developer. If anything, I think my brain is limited to holding a limited number of shortcuts for everything I use in my life. Android Studio has the perfect shortcuts for making this easier for me. Luckily, these shortcuts made it to the list of things my brain remembers.

How many times have I copied (or even cut!) text but instead of pasting, I press ⌘+V (CMD+V) again! ARGH. I used to do ⌘+Z (CMD+Z) any number of times until I get back what I wanted. That is, until I learned about ⌘+⇧+V (CMD+SHIFT+V)! This key combo shows the clipboard history, which means no more fretting. Yay!

Refactoring also mostly involves extracting variables. I have already shown [how to extract strings](http://droidista.blogspot.com.au/2015/05/stringy-strings.html) into `strings.xml`, and here I show how to extract things into methods, variables, fields, or constants.

It is fairly easy to remember them. Just combine ⌘+⌥ (CMD+OPTION) with the first letter of what you want to extract to. Time for a handy table!

| Shortcut |             | 
|----------|-------------|
| ⌘+⌥+M    | **M**ethod  |
| ⌘+⌥+V    | **V**ariable |
| ⌘+⌥+F    | **F**ield   |
| >⌘+⌥+C   | **C**onstant |

This video might do a better job of showing what I'm trying to say. Code from Chris Banes's [Cheesesquare demo](https://github.com/chrisbanes/cheesesquare).

<iframe allowfullscreen="" frameborder="0" height="315" src="https://www.youtube.com/embed/bgzPRX9zC30" width="420"></iframe>