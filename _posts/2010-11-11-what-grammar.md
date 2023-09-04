---
layout: post
title: What grammar?
date: '2010-11-11T17:57:00.003+11:00'
author: Zarah Dominguez
tags:
  - eclipse
  - grammar error
  - XML
  - android
modified_time: '2011-01-30T02:08:27.245+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-3022052835729887129
blogger_orig_url: http://www.zdominguez.com/2010/11/what-grammar.html
---

My OC side was alarmed when suddenly, my Problems view in Eclipse was filled with warnings on my XML files.  Each of my XML files had a warning with it, and that little yellow exclamation mark on the side:

```
No grammar constraints (DTD or XML schema) detected for the document
```

So how do you get rid of it?  Go to `Window > Preferences > XML > XML Files > Validation` then set `Indicate when no grammar is specified` to `Ignore>`.  Click on Apply.

Clean up your project (`Project > Clean`). 

If the problem doesn't go away, you may need to re-validate the XML files. Right click on the file then choose Validate from the popup menu.  You can also right click on the folder (such as your `res` folder) and validate from there.