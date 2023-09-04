---
layout: post
title: 'Quick Tip: Updating the location update frequency'
date: '2014-05-13T06:30:00.001+10:00'
author: Zarah Dominguez
tags:
  - gps
  - location updates
  - quick tips
  - android
modified_time: '2014-05-13T06:32:10.269+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-6572075150904575299
blogger_orig_url: http://www.zdominguez.com/2014/05/quicktip-updating-location-update.html
---

When using Google Play's Location Services and you want to change the frequency of the updates, make sure to do these in order:

```java
stopLocationUpdates();
// This method should implement mLocationClient.removeLocationUpdates()

// Set the new frequency in your location client
updateLocationClient(frequency);

startLocationUpdates();

// This method should implement mLocationClient.requestLocationUpdates()
// which means it should check for isConnected() as well!</pre>
```

If you do not remove the updates before updating the frequency, it looks like the old frequency update is still active BUT a new one is started.