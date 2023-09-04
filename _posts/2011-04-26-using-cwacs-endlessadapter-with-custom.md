---
layout: post
title: Using CWAC's EndlessAdapter with a custom adapter
date: '2011-04-26T14:49:00.010+10:00'
author: Zarah Dominguez
tags:
  - listview
  - android
  - custom adapter
  - endless adapter
modified_time: '2011-12-21T00:09:23.918+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-8222314515145947714
blogger_orig_url: http://www.zdominguez.com/2011/04/using-cwacs-endlessadapter-with-custom.html
---

In one of my projects, the app has the potential to display a very, and I mean very, long list. To minimize the loading time of the app, I limit the number of items initially included in the list and then add to it as the user scrolls down.

For this purpose, [Mark Murphy](http://commonsware.com/)'s [EndlessAdapter](https://github.com/commonsguy/cwac-endless) works wonders. I was trying to make it work with a `CursorAdapter` though, but due to time constraints, I was not able to continue with my experiment.

And then I found out that some of the items in my list are HTML-formatted. Huh. So I have to have a custom adapter to override `getView()`. I patterned my code on the demo included in the `EndlessAdapter` project and I was at a loss. Maybe it's because I just came from a vacation and my mind refuses to work. Hmmm.

To cut a long and arduous journey short, I was able to figure it out. Here's a sample activity that displays a list of countries from an array. To illustrate using a custom adapter, I add the list item number when setting the item text.

I will discuss the pertinent parts of the code in detail.

```java
private void init() {
    LIST_SIZE = COUNTRIES.length;
    for (int i=0; i<=BATCH_SIZE; i++){
        countriesSub.add(COUNTRIES[i]);
    }
    
    setLastOffset(BATCH_SIZE);
    displayList(countriesSub);
}
```

In this part of the code, I get the first 10 items in the list as the initial contents of the list. Of course, our current list is small and is peanuts for `ListView`. This is just to illustrate my point. In my app, I initially load 2,000 items. I also make sure to remember where I am in my source list. In my case, it is the offset in the original array. This might also be a row in the DB, or the ID in an RSS feed.

The `ArrayList` `countriesSub` holds the items that are currently in my list. As the user goes through the list, this array will grow.

To display my list, I set an instance of `DemoAdapter` as my list adapter. `DemoAdapter` inherits from `EndlessAdapter` and in its constructor, I give it an instance of my custom `ArrayAdapter`.

```java
@Override
public View getView(int position,View convertView,ViewGroup parent){
        ViewHolder holder;

        if(convertView == null) {
            LayoutInflater inflater = (LayoutInflater)EndlessCustom.this.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
            convertView = inflater.inflate(android.R.layout.simple_list_item_1,null);
            holder = new ViewHolder();
            holder.word = (TextView)convertView.findViewById(android.R.id.text1);
            convertView.setTag(holder);
        } else {
            holder=(ViewHolder)convertView.getTag();
        }

        holder.word.setText(String.valueOf(position+1)+". "+countriesSub.get(position));
        return convertView;
        }
```

The important part in my custom `ArrayAdapter` is the `getView()` method. In this method, I tell the adapter to not simply display the `.toString()` value of the item, but to add a number before it.

Notice that I use the `ViewHolder` pattern [as illustrated in the Android Developers site](http://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/view/List14.html).
```java
DemoAdapter() {
    super(new CustomArrayAdapter(EndlessCustom.this, android.R.layout.simple_list_item_1, android.R.id.text1, countriesSub));
    
    rotate = new RotateAnimation(0f, 360f, 
        Animation.RELATIVE_TO_SELF, 0.5f, 
        Animation.RELATIVE_TO_SELF, 0.5f);
    rotate.setDuration(600);
    rotate.setRepeatMode(Animation.RESTART);
    rotate.setRepeatCount(Animation.INFINITE);
}
```

So now we come to the `EndlessAdapter` part. In the constructor, I pass into it an instance of my custom `ArrayAdapter`, indicating the source of the list, the layout for each row, and the `TextView` ID from the layout. I also instantiated the animation that will be used while the list is loading the additional items.

```java
@Override
protected boolean cacheInBackground() {
    tempList.clear();
    int lastOffset = getLastOffset();
    if (lastOffset < LIST_SIZE) {
        int limit = lastOffset + BATCH_SIZE;
        for(int i=(lastOffset+1); (i<=limit && i<LIST_SIZE); i++){
            tempList.add(COUNTRIES[i]);
        } 
        
        setLastOffset(limit);
        
        if (limit < LIST_SIZE) {
            return true;
        } else {
            return false;
        }
    } else  {
        return false;
    }
}
```

We have to override `cacheInBackground()` for `EndlessAdapter` to work. Here we do the heavy lifting like querying the server, reading from the DB, etc. Here, I load the next 10 items from the original list and put them in a temporary `ArrayList`. I also check whether I have loaded all the data, and if so, tell the `EndlessAdapter` to not show the extra row at the bottom. I do this by returning `false` from the method.

```java
@Override
protected void appendCachedData() {
    @SuppressWarnings("unchecked")
    ArrayAdapter<String> arrAdapterNew = (ArrayAdapter<String>)getWrappedAdapter();
    
    int listLen = tempList.size();
    for(int i=0; i<listLen; i++){
        arrAdapterNew.add(tempList.get(i));
    }
}
```
Finally, I add the newly retrieved rows to my current list. And that's it.

The complete Java file for this activity follows:
<details>
<summary>Sample code</summary>

```java
package com.test.example;

import java.util.ArrayList;
import java.util.List;

import android.app.ListActivity;
import android.content.Context;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.view.animation.Animation;
import android.view.animation.RotateAnimation;
import android.widget.ArrayAdapter;
import android.widget.TextView;

import com.commonsware.cwac.endless.EndlessAdapter;

public class EndlessCustom extends ListActivity {

 static int LIST_SIZE;
 private int mLastOffset = 0;
 
 static final int BATCH_SIZE = 10;
 
 ArrayList&lt;String&gt; countriesSub = new ArrayList&lt;String&gt;();
 
 @Override
 public void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.lib_activity_dictionary);
  init();
 }

 private void init() {
  LIST_SIZE = COUNTRIES.length;  
  for (int i=0; i&lt;=BATCH_SIZE; i++){
   countriesSub.add(COUNTRIES[i]);   
  }
  setLastOffset(BATCH_SIZE);
  displayList(countriesSub);
 }

 private void setLastOffset(int i) {
  mLastOffset = i;  
 }
 
 private int getLastOffset(){
  return mLastOffset;
 }

 private void displayList(ArrayList&lt;String&gt; countriesSub) {  
  setListAdapter(new DemoAdapter());
 }

 private class CustomArrayAdapter extends ArrayAdapter&lt;String&gt;{

  public CustomArrayAdapter(Context context, int resource,
    int textViewResourceId, List&lt;String&gt; objects) {
   super(context, resource, textViewResourceId, objects);
  }  

  @Override
  public View getView(int position, View convertView, ViewGroup parent) {

   ViewHolder holder;   

   if(convertView==null){
    LayoutInflater inflater = (LayoutInflater)
    EndlessCustom.this.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    convertView = inflater.inflate(android.R.layout.simple_list_item_1, null);
    holder = new ViewHolder();
    holder.word = (TextView)convertView.findViewById(android.R.id.text1);

    convertView.setTag(holder);
   } else{
    holder=(ViewHolder)convertView.getTag();
   }

   holder.word.setText(String.valueOf(position+1) + ". " + countriesSub.get(position));
   return convertView;
  }

  public class ViewHolder{
   public TextView word;        
  }  
 }

 class DemoAdapter extends EndlessAdapter {
  private RotateAnimation rotate=null;
  ArrayList&lt;String&gt; tempList = new ArrayList&lt;String&gt;();
  
  DemoAdapter() {
   super(new CustomArrayAdapter(EndlessCustom.this, 
     android.R.layout.simple_list_item_1, android.R.id.text1, countriesSub));

   rotate=new RotateAnimation(0f, 360f, Animation.RELATIVE_TO_SELF,
     0.5f, Animation.RELATIVE_TO_SELF,
     0.5f);
   rotate.setDuration(600);
   rotate.setRepeatMode(Animation.RESTART);
   rotate.setRepeatCount(Animation.INFINITE);
  }

  @Override
  protected View getPendingView(ViewGroup parent) {
   View row=getLayoutInflater().inflate(R.layout.row, null);

   View child=row.findViewById(android.R.id.text1);
   child.setVisibility(View.GONE);
   child=row.findViewById(R.id.throbber);
   child.setVisibility(View.VISIBLE);
   child.startAnimation(rotate);

   return(row);
  }

  @Override
  protected boolean cacheInBackground() {
   tempList.clear();
   int lastOffset = getLastOffset();
   if(lastOffset &lt; LIST_SIZE){
    int limit = lastOffset + BATCH_SIZE;
    for(int i=(lastOffset+1); (i&lt;=limit &amp;&amp; i&lt;LIST_SIZE); i++){
     tempList.add(COUNTRIES[i]);
    }    
    setLastOffset(limit);
    
    if(limit&lt;LIST_SIZE){
     return true;
    } else {
     return false;
    }
   } else  {
    return false;
   }
  }


  @Override
  protected void appendCachedData() {

   @SuppressWarnings("unchecked")
   ArrayAdapter&lt;String&gt; arrAdapterNew = (ArrayAdapter&lt;String&gt;)getWrappedAdapter();

   int listLen = tempList.size();
   for(int i=0; i&lt;listLen; i++){
    arrAdapterNew.add(tempList.get(i));
   }
  }
 }
 
 
 static final String[] COUNTRIES = new String[] {
        "Afghanistan", "Albania", "Algeria", "American Samoa", "Andorra", 
        "Angola", "Anguilla", "Antarctica", "Antigua and Barbuda", "Argentina",
        "Armenia", "Aruba", "Australia", "Austria", "Azerbaijan",
        "Bahrain", "Bangladesh", "Barbados", "Belarus", "Belgium",
        "Belize", "Benin", "Bermuda", "Bhutan", "Bolivia",
        "Bosnia and Herzegovina", "Botswana", "Bouvet Island", "Brazil", "British Indian Ocean Territory",
        "British Virgin Islands", "Brunei", "Bulgaria", "Burkina Faso", "Burundi",
        "Cote d'Ivoire", "Cambodia", "Cameroon", "Canada", "Cape Verde",
        "Cayman Islands", "Central African Republic", "Chad", "Chile", "China",
        "Christmas Island", "Cocos (Keeling) Islands", "Colombia", "Comoros", "Congo",
        "Cook Islands", "Costa Rica", "Croatia", "Cuba", "Cyprus", "Czech Republic",
        "Democratic Republic of the Congo", "Denmark", "Djibouti", "Dominica", "Dominican Republic",
        "East Timor", "Ecuador", "Egypt", "El Salvador", "Equatorial Guinea", "Eritrea",
        "Estonia", "Ethiopia", "Faeroe Islands", "Falkland Islands", "Fiji", "Finland",
        "Former Yugoslav Republic of Macedonia", "France", "French Guiana", "French Polynesia",
        "French Southern Territories", "Gabon", "Georgia", "Germany", "Ghana", "Gibraltar",
        "Greece", "Greenland", "Grenada", "Guadeloupe", "Guam", "Guatemala", "Guinea", "Guinea-Bissau",
        "Guyana", "Haiti", "Heard Island and McDonald Islands", "Honduras", "Hong Kong", "Hungary",
        "Iceland", "India", "Indonesia", "Iran", "Iraq", "Ireland", "Israel", "Italy", "Jamaica",
        "Japan", "Jordan", "Kazakhstan", "Kenya", "Kiribati", "Kuwait", "Kyrgyzstan", "Laos",
        "Latvia", "Lebanon", "Lesotho", "Liberia", "Libya", "Liechtenstein", "Lithuania", "Luxembourg",
        "Macau", "Madagascar", "Malawi", "Malaysia", "Maldives", "Mali", "Malta", "Marshall Islands",
        "Martinique", "Mauritania", "Mauritius", "Mayotte", "Mexico", "Micronesia", "Moldova",
        "Monaco", "Mongolia", "Montserrat", "Morocco", "Mozambique", "Myanmar", "Namibia",
        "Nauru", "Nepal", "Netherlands", "Netherlands Antilles", "New Caledonia", "New Zealand",
        "Nicaragua", "Niger", "Nigeria", "Niue", "Norfolk Island", "North Korea", "Northern Marianas",
        "Norway", "Oman", "Pakistan", "Palau", "Panama", "Papua New Guinea", "Paraguay", "Peru",
        "Philippines", "Pitcairn Islands", "Poland", "Portugal", "Puerto Rico", "Qatar",
        "Reunion", "Romania", "Russia", "Rwanda", "Sqo Tome and Principe", "Saint Helena",
        "Saint Kitts and Nevis", "Saint Lucia", "Saint Pierre and Miquelon",
        "Saint Vincent and the Grenadines", "Samoa", "San Marino", "Saudi Arabia", "Senegal",
        "Seychelles", "Sierra Leone", "Singapore", "Slovakia", "Slovenia", "Solomon Islands",
        "Somalia", "South Africa", "South Georgia and the South Sandwich Islands", "South Korea",
        "Spain", "Sri Lanka", "Sudan", "Suriname", "Svalbard and Jan Mayen", "Swaziland", "Sweden",
        "Switzerland", "Syria", "Taiwan", "Tajikistan", "Tanzania", "Thailand", "The Bahamas",
        "The Gambia", "Togo", "Tokelau", "Tonga", "Trinidad and Tobago", "Tunisia", "Turkey",
        "Turkmenistan", "Turks and Caicos Islands", "Tuvalu", "Virgin Islands", "Uganda",
        "Ukraine", "United Arab Emirates", "United Kingdom",
        "United States", "United States Minor Outlying Islands", "Uruguay", "Uzbekistan",
        "Vanuatu", "Vatican City", "Venezuela", "Vietnam", "Wallis and Futuna", "Western Sahara",
        "Yemen", "Yugoslavia", "Zambia", "Zimbabwe"
    };
}
```
</details>

  
If you know of a more efficient way to do this, please do not hesitate to let me know! :)