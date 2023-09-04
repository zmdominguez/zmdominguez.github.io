---
layout: post
title: I Can Haz Internetz!
date: '2012-10-28T01:34:00.001+11:00'
author: Zarah Dominguez
tags:
  - connectivity
  - notification
  - android
modified_time: '2012-10-28T01:58:55.173+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-4512138566188909306
blogger_orig_url: http://www.zdominguez.com/2012/10/i-can-haz-internetz.html
---

Last week, I was exploring connectivity monitoring and came up with a small app for demo. The app listens for connectivity changes and sends a notification to the user informing them of the change.

First off, create a class (I named mine `ConnectivityUpdate.java`) and make it extend `BroadcastReceiver`. This class should do what you want when notified of connectivity changes. You will be asked to implement the `onReceive()` method. Now here's what I want to happen when the system notifies me of changes:
1. Check what kind of change I am being notified of: did I get Internet? Or did I lose Internet?
2. Compare this to what connectivity I had before the notification so I can inform the user (and app-wise do whatever it is I'm supposed to do like start syncing or stop syncing).

In my implementation, this is done as:
```java
@Override
public void onReceive(Context context, Intent intent) {
     
    boolean hasInternet = false;
    Log.d(LOG_TAG, "Received broadcast!");
     
    // Do I have Internet?
    if(intent.getAction().equals(ConnectivityManager.CONNECTIVITY_ACTION)) {
        hasInternet = (new NDUtils()).hasInternet(context);
    }
    
    // Did I use to have Internet?
    SharedPrefsManager prefs = new SharedPrefsManager();
    boolean prevValue = prefs.hasNetwork(context);
    
    if(prevValue != hasInternet){
        // Remember what we have now
        (new SharedPrefsManager()).saveNetworkState(context, hasInternet);
        sendNotification(hasInternet, context);
    }
}
```

The actual checking if we have Internet or not is done by a utility class: 
```java
public boolean hasInternet(Context context) {
    
    NetworkInfo networkInfo = (NetworkInfo) ((ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE)).getActiveNetworkInfo();
    
    // Check if we are connected to Wifi or mobile internet
    // AND if we can actually send data
    if(networkInfo != null && 
        (networkInfo.getType() == ConnectivityManager.TYPE_WIFI ||
        networkInfo.getType() == ConnectivityManager.TYPE_MOBILE) &&
        networkInfo.isConnected()) {
        return true;
    }  
    
    return false;
}
```

The Android [javadoc on `getActiveNetworkInfo()`](http://developer.android.com/reference/android/net/ConnectivityManager.html#getActiveNetworkInfo()) says it may return `null`, so we check first to avoid `NullPointerException`s, then we check if we have WiFi or mobile internet, THEN (and this is important) we check if we can send data through this network.

You can also refine this filter more by checking the type of mobile network the user has (we want 3G or faster) or by checking if the user is roaming (user should not be roaming). Here is a more complete method describing what I just said:
```java
public boolean hasInternet(Context context){
    
    NetworkInfo networkInfo = (NetworkInfo)((ConnectivityManager)context.getSystemService(Context.CONNECTIVITY_SERVICE)).getActiveNetworkInfo();
    TelephonyManager telephony = (TelephonyManager)context.getSystemService(Context.TELEPHONY_SERVICE);
    
    if(networkInfo != null && networkInfo.isConnected() &&
        (networkInfo.getType() == ConnectivityManager.TYPE_WIFI ||
        (networkInfo.getType() == ConnectivityManager.TYPE_MOBILE && 
        networkInfo.getSubtype() > TelephonyManager.NETWORK_TYPE_UMTS &&
        !telephony.isNetworkRoaming()))){

        return true;
    }
    
    return false;
}
```

I then compare the returned value of the utility method to the last value saved in my preferences file. That part of the code is straight up saving/getting a boolean value from a `SharedPreferences` file.

I then inform the user of the network change through a notification. When the user clicks on the expanded notification, I want to open the app. The app can then display details about the network, but right now what my sample app does is simply show the boolean value returned by the utility method. Anyway, here's how I send the notification:

```java
private void sendNotification(boolean hasInternet, Context context) {
    
        NotificationManager notificationManager = (NotificationManager)context.getSystemService(Context.NOTIFICATION_SERVICE);
        
        // The notification details seen by the user when the drawer is pulled down
        String notificationTitle = "Change in data connection!";
        String notificationText = "We have internet?: " + (hasInternet ? "YES!" : "NO :(");
        
        // The icon and the text to be displayed in the notification area
        Notification myNotification = new Notification(R.drawable.ic_launcher, "Broadcast received!", System.currentTimeMillis());
        
        // Create a new intent to launch my app
        Intent myIntent = new Intent(context.getApplicationContext(), NetworkDetector.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(context.getApplicationContext(), 0, myIntent, Intent.FLAG_ACTIVITY_NEW_TASK);
        
        // Send that notification! MY_NOTIFICATION_ID is a value greater than 0.
        myNotification.setLatestEventInfo(context, 
            notificationTitle,
            notificationText,
            pendingIntent);
        notificationManager.notify(MY_NOTIFICATION_ID, myNotification);
}
```


So how do we tell the OS that we have made this receiver and please notify us when the connectivity changes, Mister Android please? We create a broadcast receiver in the app's manifest! We listen to both `CONNECTIVITY_CHANGE` and WiFi `STATE_CHANGE`. The value for `android:name` is the name of the class containing the receiver implementation.
```xml
<receiver 
    android:name=".util.ConnectivityUpdate"
    android:enabled="true" 
    android:exported="true"
    android:label="ConnectivityActionReceiver">
    
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
        <action android:name="android.net.wifi.STATE_CHANGE" />
    </intent-filter>
    
</receiver>
```

And don't forget to add the permissions to read the network state:
```xml
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

The complete source code for this sample app [is in github](https://github.com/zmdominguez/network-detect-notify).