---
layout: post
title: Enabling/disabling menu items on the fly
date: '2011-01-30T01:12:00.005+11:00'
author: Zarah Dominguez
tags:
  - menu
  - android
modified_time: '2011-02-02T22:00:02.055+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-427842400213856168
blogger_orig_url: http://www.zdominguez.com/2011/01/enablingdisabling-menu-items-on-fly.html
---

In one of my applications, I want to disable some menu entries if the database is not valid or is not present at all. To do that, I make use of the `onPrepareOptionsMenu`() API.

Let's say the user is on the <span style="font-style:italic;">Quiz </span>activity.  When the user presses the <span style="font-style:italic;">MENU </span>button, the <span style="font-style:italic;">Quiz </span>item should not be there anymore; and until the application validates the presence of a database, I want the <span style="font-style:italic;">Books </span>item to be unclickable.

So I inflate my menu as usual from the XML file:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.my_menu, menu);
    
    removeSomeItems(menu);
    return true;
}

private void removeSomeItems(Menu menu){
    MenuItem books = menu.findItem(R.id.menu_books);
    books.setEnabled(false);  // I want this item to be available eventually
    menu.removeItem(R.id.menu_quiz); // I want this item to be unavailablefor this activity
}
```

You have to remember though, that this method is called only ONCE, the very first time the user opens the menu.

The activity does its business as usual.  When I am done checking if the database is there already, I want the user to be able to use the _Books_ item.  I have a `boolean` field that I set when the database is validated, and so now I can enable the menu.

```java
@Override
public boolean onPrepareOptionsMenu (Menu menu){
    super.onPrepareOptionsMenu(menu);
    if(mIsDbValid){
        MenuItem books = menu.findItem(R.id.menu_books);
        books.setEnabled(true);
    }
    return true;
}
```


You can change the contents of the menu as often as you want using `onPrepareOptionsMenu`() because that method is called each and every time the user presses the _MENU_ button.

And that's it!