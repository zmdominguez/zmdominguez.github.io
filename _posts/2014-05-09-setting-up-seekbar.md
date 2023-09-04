---
layout: post
title: 'Setting up the SeekBar '
date: '2014-05-09T01:22:00.000+10:00'
author: Zarah Dominguez
tags:
  - seekbar
  - android
modified_time: '2014-05-09T01:22:18.367+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-4450893282935924413
blogger_orig_url: http://www.zdominguez.com/2014/05/setting-up-seekbar.html
---

So we want to use the [`SeekBar`](http://developer.android.com/reference/android/widget/SeekBar.html). We want the minimum value to be 10 and the maximum value to be 100, and it should increment by 10.

Thing is, `SeekBar` by default always starts at 0, and the increment is always an `int`. It is definitely possible to get what we want, but we need to do some simple math first.

Compute how many increments you will need from your minimum up to your maximum:
```shell
numberOfIncrements = maximum - minimum = 90
```

Then divide it by the amount of each increment we want:
```shell
seekBarMaximum = numberOfIncrements / 10 = 9
```

This means we should set up the SeekBar to have max = 9 and increment = 1. Then in our code, we have to figure out how to get the actual progress that we want.

```java
SeekBar.OnSeekBarChangeListener mSeekbarListener = new OnSeekBarChangeListener() {
    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {
        /* Do nothing*/ 
    }
    
    @Override
    public void onStartTrackingTouch(SeekBar seekBar) {
        /* Do nothing*/ 
    }
    
    @Override
    public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser){
        mProgressDisplay.setText("Seekbar is at: " + getProgressToDisplay(progress));
        }
    
    };
    
    private String getProgressToDisplay(int progress) {
        // We are multiplying by 10 since it is our actual increment
        int actualProgress = (progress + 1) * 10;
        return String.valueOf(actualProgress);
    }
```

Another example:
```shell
minimum = 1, maximum = 10, increment = 0.5

numberOfIncrements = 9
seekBarMaximum = 18
```

In this case, the contents of `getProgressToDisplay()` will change since the increment is not a whole number.
```java
private String getProgressToDisplay(int progress) {
    float actualProgress = (progress + 1) - (progress * 0.5f);
    return String.valueOf(actualProgress);
}
```
