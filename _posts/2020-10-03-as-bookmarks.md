---
layout: post
title: "Easy Navigation with Bookmarks"
tags:
    - android studio
---

One of the things I find most challenging when learning a new part of a codebase is navigating through the data flow. In some areas of our app, following RX streams can be particularly... draining.

After following all the subscribers and the observers and how things get from one place to the next, going back and doing everything all over again when I need to debug something gets a bit tedious.

Sure I can `⌘+[` / `⌘+]` but sometimes there are things in the history that I don't really care for (I'm bound to eventually get sidetracked for sure). This is when [bookmarks](https://www.jetbrains.com/help/idea/navigating-through-the-source-code.html#use_bookmarks) shine for me.

Bookmarks, as the name implies, are markers that make it easy to go back to a particular line of code. Bookmarks can be set on the line where the caret is via `F3` or by right-clicking on the gutter and selecting "Set Bookmark".
<center>
    <a href="https://imgur.com/jKpj2TG"><img src="https://i.imgur.com/jKpj2TG.png?1" title="source: imgur.com" /></a><br />
    <small>Adding a bookmark</small>
</center>

Hit `⌘+F3` to open the [Bookmarks dialog](https://www.jetbrains.com/help/idea/bookmarks-dialog.html) to see all available options for managing bookmarks. Here we can edit descriptions for each bookmark to tell them apart, sort or re-order bookmarks, remove an existing bookmark, or check the code preview.

<center>
    <a href="https://imgur.com/OI937ks"><img src="https://i.imgur.com/OI937ks.png" title="source: imgur.com" /></a><br />
    <small>The Bookmarks dialog</small>
</center>

There is no default keyboard shortcut assigned for moving through the bookmarks in order, but assigning one is pretty straightforward. Go to `Preferences > Keymap > Main Menu > Navigate > Bookmarks`, right click on "Next Bookmark" or "Previous Bookmark" and choose "Add Keyboard Shortcut".
<center>
    <a href="https://imgur.com/aRtGHre"><img src="https://i.imgur.com/aRtGHre.png" title="source: imgur.com" /></a><br />
    <small>Assign a keyboard shortcut</small>
</center>
I have chosen to use `⌘+Shift+F3` (go to next bookmark) / `^+Shift+F3` (go to previous bookmark) for my shortcuts.

I have lost count of the times that I _know_ I have seen something before and just can't remember where. Nowadays when I am trying to learn an unfamiliar part of our codebase, I have made it a habit to drop bookmarks on the most relevant parts of the code. Definitely saved me a lot of frustration!

