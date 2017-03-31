---
layout: post
title: 'More plurals: decimal values'
date: '2010-09-15T14:45:00.012+10:00'
author: Zarah Dominguez
tags:
- plurals
- strings
- android
- decimal
modified_time: '2010-10-26T20:11:25.005+11:00'
thumbnail: http://4.bp.blogspot.com/_p3DeelS9Gp8/TJBWSMg5XkI/AAAAAAAAA28/JHLcPbv0wQQ/s72-c/plurals_decimal_less1.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-8977124023305288553
blogger_orig_url: http://www.zdominguez.com/2010/09/more-plurals-decimal-values.html
---

In my [previous post](http://droidista.blogspot.com/2010/09/string-pluralization.html), I showed you how to set string plurals. If you noticed, the methods to get the plurals strings only accept `int`s. What if (like me) you want to display a decimal value? I am getting my raw value from a progress bar with a range of 1-10, with 0.1 increments.

First, to display decimal values, I set my plurals string to display a float value.

```xml
<item quantity="other">Progress is at %.1f units.</item>
```

And then I devised a way to set the quantity based on the value of the progress bar (`ProgressBar.getProgress()` returns an `int`).

```java
public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {

    // get quantity for plurals string
    int quantity = setQuantity(progress);

    // convert the actual progress to float
    float floatProgress = convertProgress(progress);

    // get the actual string and replace formatting with the float value
    String currentProgress = getResources()
         .getQuantityString(R.plurals.seekBarProgress, // get the plurals
         quantity, // set the quantity
         floatProgress); // format arguments

    // set text to display
    TextView displayProgress = (TextView)findViewById(R.id.prog_text);
    displayProgress.setText(currentProgress);

}

/**
* Use this method to see if we will use the singular or plural string.
*
* @param progress
* @return the value to set in getQuantityString()
*/
private int setQuantity(int progress){
   int quantity;

   if (((progress%10) == 0) && ((progress/10) == 1)){
       quantity = 1;
   } else {
       quantity = 2;
   }

   return quantity;
}

/**
* Use this value to get the *actual* value to display.
*
* @param progress actual progress value from 0 to 100
* @return the float value from 0.0 to 10.0
*/
private Float convertProgress(int progress){
   return ((Float.valueOf(String.valueOf(progress)))/(float)10);
}
```

So you see, it's quite long-winded. Here are some screen shots of the results:

![](http://4.bp.blogspot.com/_p3DeelS9Gp8/TJBWSMg5XkI/AAAAAAAAA28/JHLcPbv0wQQ/s320/plurals_decimal_less1.png) ![](http://2.bp.blogspot.com/_p3DeelS9Gp8/TJBWR4FQZjI/AAAAAAAAA20/LWlDCWAVfEk/s320/plurals_decimal_1.png) [![](http://2.bp.blogspot.com/_p3DeelS9Gp8/TJBWSZX6GaI/AAAAAAAAA3E/prxdkDjhaEk/s320/plurals_decimal_more1.png)](http://2.bp.blogspot.com/_p3DeelS9Gp8/TJBWSZX6GaI/AAAAAAAAA3E/prxdkDjhaEk/s1600/plurals_decimal_more1.png)

Different values for the unit value
