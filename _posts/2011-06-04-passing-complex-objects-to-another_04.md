---
layout: post
title: Passing complex objects to another Activity
date: '2011-06-04T02:21:00.007+10:00'
author: Zarah Dominguez
tags:
  - extras
  - intents
  - complex values
  - android
modified_time: '2012-02-20T14:13:34.828+11:00'
thumbnail: http://4.bp.blogspot.com/-khlSEQbj8TI/TekORjI-cPI/AAAAAAAAA5U/yV1IRvPIIJI/s72-c/complex_object_screen.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-2840671726122369065
blogger_orig_url: http://www.zdominguez.com/2011/06/passing-complex-objects-to-another_04.html
---

Several months ago, I was faced with a problem of passing a complex object to another Activity. There are several ways of doing this:
- "Deconstructing" the complex object to simple data types and passing them as extras through `putExtra()`
- Making the object `Parcelable`
- Making the object `Serializable`

I don't really understand the concepts behind an object being `Parcelable` or `Serializable`, so I was not comfortable using that approach. Deconstructing the object, on the other hand, is easier but it is quite hard to manage as the number of fields in the object increases.

And then I came across another solution: using `Bundle`s. In this post, I will try to explain how to do that.

In this sample project, we want to pass an instance of an `AttendeeObject` from our main activity to another. To better illustrate how we can use this method for different data types, an `AttendeeObject` will hold three pieces of information: a `String` name, an `Integer` age, and a `boolean` indicating the person's presence.

First, we construct the `AttendeeObject` like any old POJO. Create all the fields you want the object to contain and generate getters and setters for each. We then create the methods we would need to be able to pass, and subsequently receive, this object between activities.

To pass an `AttendeeObject`, we would need to put it in a `Bundle` and pass it using [Intent#putExtra(String key, Bundle bundle)](http://developer.android.com/reference/android/content/Intent.html#putExtra(java.lang.String, android.os.Bundle)).
```java
public Bundle bundleAttendee(AttendeeObject attendee){
     Bundle bundle = new Bundle();
     bundle.putString(NAME, attendee.getName());
     bundle.putInt(AGE, attendee.getAge());
     bundle.putBoolean(PRESENCE, attendee.isPresent());
  
     return bundle;
}
```
Here, we simply put into a `Bundle` the values in the object that we want to pass.

Next, we should be able to convert this bundle back to an `AttendeeObject` in the receiving `Activity`. This task is done by the following method:
```java
public AttendeeObject unBundleAttendee(Bundle attendeeBundle){
     AttendeeObject attendee = new AttendeeObject();
     attendee.setName(attendeeBundle.getString(NAME));
     attendee.setAge(attendeeBundle.getInt(AGE));
     attendee.setIsPresent(attendeeBundle.getBoolean(PRESENCE));
  
     return attendee;
}
```

Now, we are ready to use this in our application. I created a simple app that asks the user for the three values we would need to populate `AttendeeObject`.
<a href="http://4.bp.blogspot.com/-khlSEQbj8TI/TekORjI-cPI/AAAAAAAAA5U/yV1IRvPIIJI/s1600/complex_object_screen.png"><img alt="" border="0" id="BLOGGER_PHOTO_ID_5614034105147486450" src="http://4.bp.blogspot.com/-khlSEQbj8TI/TekORjI-cPI/AAAAAAAAA5U/yV1IRvPIIJI/s200/complex_object_screen.png" style="cursor: hand; cursor: pointer; display: block; height: 200px; margin: 0px auto 10px; text-align: center; width: 134px;" /></a>

I won't discuss the details of creating the layout file, but it might be worthy to note that the possible values for the Spinner are just "Yes" and "No".

When the user clicks on the "Create Attendee" button, the application creates a new `AttendeeObject` based on the values in the form and adds this to an `ArrayList` of `AttendeeObject`s.
```java
private void onCreateAttendee() {
     // Create a new AttendeeObject
     AttendeeObject attendee = new AttendeeObject();
     attendee.setName(mNameText.getText().toString());
     attendee.setAge(Integer.valueOf(mAgeText.getText().toString()));
     attendee.setIsPresent(mCurrentPresence);
  
     // You can then do whatever you want with this object
     // like adding it to an ArrayList or saving it to a database
     mAttendees.add(attendee);
  
     Toast.makeText(ComplexObjectDemo.this, "Attendee added!", Toast.LENGTH_SHORT).show();
}
```

When the user clicks on the "Submit Attendee" button, the app "packages" the chosen value into a `Bundle` and sets it as an `extra` to the `Intent`. For illustration purposes, the application always gets the second `AttendeeObject` in the `ArrayList`.

```java
private void onSubmitObject() {
     Intent intent = new Intent(ComplexObjectDemo.this, ComplexObjectReceiverDemo.class);

     // For simplicity, we will always get the second value
     Bundle value = createAttendeeBundle(1);

     // In a "real" application, the Key should be defined in a constants file
     intent.putExtra("test.complex.attendee", value);
  
     startActivity(intent);
}

private Bundle createAttendeeBundle(int index) {
     // Here, mAttendee is an instance of AttendeeObject
     // and mAttendees is an ArrayList holding all the created attendees
     return mAttendee.bundleAttendee(mAttendees.get(index));
}
```

Upon receiving the `Intent`, the new activity simply displays the values in the `AttendeeObject` passed into it. Here's how we can "unbundle" the `Bundle`:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.activity_complex_demo_receiver);

     // Get the extras passed to this Activity        
     setUpViews(getIntent().getExtras());        
}

private void setUpViews(Bundle extras) {
     // Again, the key should be in a constants file!
     Bundle extraAttendee = extras.getBundle("test.complex.attendee");
 
     AttendeeObject attendee = new AttendeeObject();

     // Deconstruct the Bundle into the AttendeeObject
     attendee = attendee.unBundleAttendee(extraAttendee);

     // The AttendeeObject fields are now populated, so we set the texts
     ((TextView) findViewById(R.id.attendee_age_value))
            .setText(String.valueOf(attendee.getAge()));
     ((TextView) findViewById(R.id.attendee_name_value))
            .setText(attendee.getName());
     ((TextView) findViewById(R.id.attendee_presence_value))
            .setText(String.valueOf(attendee.isPresent()));
}
```

Here's how the receiving activity looks like:<a href="http://1.bp.blogspot.com/-KqcJ4DHaGsQ/TekU5ma_CpI/AAAAAAAAA5c/2fOBbdkiGeI/s1600/complex_object_result.png"><img alt="" border="0" id="BLOGGER_PHOTO_ID_5614041390292863634" src="http://1.bp.blogspot.com/-KqcJ4DHaGsQ/TekU5ma_CpI/AAAAAAAAA5c/2fOBbdkiGeI/s200/complex_object_result.png" style="cursor: hand; cursor: pointer; display: block; height: 200px; margin: 0px auto 10px; text-align: center; width: 134px;" /></a>

Here's the complete `AttendeeObject` class.

<details>
<summary>`AttendeeObject` code</summary>

```java
package droidista.example;

import android.os.Bundle;

public class AttendeeObject {

    public static final String NAME = "attendee.name";
    public static final String AGE = "attendee.age";
    public static final String PRESENCE = "attendee.presence";


    private String mName;
    private int mAge;
    private boolean mIsPresent;

    public AttendeeObject() {

    }

    public Bundle bundleAttendee(AttendeeObject attendee) {
        Bundle bundle = new Bundle();
        bundle.putString(NAME, attendee.getName());
        bundle.putInt(AGE, attendee.getAge());
        bundle.putBoolean(PRESENCE, attendee.isPresent());

        return bundle;
    }

    public AttendeeObject unBundleAttendee(Bundle attendeeBundle) {
        AttendeeObject attendee = new AttendeeObject();
        attendee.setName(attendeeBundle.getString(NAME));
        attendee.setAge(attendeeBundle.getInt(AGE));
        attendee.setIsPresent(attendeeBundle.getBoolean(PRESENCE));

        return attendee;
    }

    public String getName() {
        return mName;
    }

    public void setName(String name) {
        this.mName = name;
    }

    public int getAge() {
        return mAge;
    }

    public void setAge(int age) {
        this.mAge = age;
    }

    public boolean isPresent() {
        return mIsPresent;
    }

    public void setIsPresent(boolean isPresent) {
        this.mIsPresent = isPresent;
    }
}
```
</details>

And our main activity:

<details>
<summary>`MainActivity` code</summary>

```java
package droidista.example;

import java.util.ArrayList;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Spinner;
import android.widget.Toast;
import android.widget.AdapterView.OnItemSelectedListener;

public class ComplexObjectDemo extends Activity implements OnClickListener {
 
 private Button mGoToNextIntent, mCreateAttendee, mClearFields;
 private EditText mNameText, mAgeText;
 private Spinner mPresence;
 
 AttendeeObject mAttendee = new AttendeeObject();
 
 private boolean mCurrentPresence;
 
 private ArrayList&lt;AttendeeObject&gt; mAttendees = new ArrayList&lt;AttendeeObject&gt;();
 
 @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_complex_demo);
        
        setUpViews();
 }

 private void setUpViews() {
  setUpSpinner();  
     
  mNameText = (EditText)findViewById(R.id.name_text_input);
  mAgeText = (EditText)findViewById(R.id.age_text_input);  
  
  mGoToNextIntent = (Button)findViewById(R.id.btn_submit_object);
  mGoToNextIntent.setOnClickListener(this);
  
  mCreateAttendee = (Button)findViewById(R.id.btn_create_attendee);
  mCreateAttendee.setOnClickListener(this);
  
  mClearFields = (Button) findViewById(R.id.btn_clear_fields);
  mClearFields.setOnClickListener(this);
 } 
 
 private void setUpSpinner() {
  mPresence = (Spinner) findViewById(R.id.presence_spinner);
     ArrayAdapter&lt;CharSequence&gt; adapter = ArrayAdapter.createFromResource(
             this, R.array.is_present_values, android.R.layout.simple_spinner_item);
     adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
     mPresence.setAdapter(adapter);
 }
 
 
 private Bundle createAttendeeBundle(int index) {
  // Here, mAttendee is an instance of AttendeeObject
  // and mAttendees is an ArrayList holding all the created attendees
  return mAttendee.bundleAttendee(mAttendees.get(index));
 }

 public void onClick(View v) {
  switch(v.getId()){
  case R.id.btn_create_attendee:
   onCreateAttendee();
   break;
  case R.id.btn_submit_object:
   onSubmitObject();  
   break;
  case R.id.btn_clear_fields:
   onClearFields();
   break;
  }
 }

 private void onClearFields() {
  mAgeText.setText("");
  mNameText.setText("");
  mPresence.setSelection(0);
 }

 private void onSubmitObject() { 

  // For simplicity, we will always get the second value
  Bundle value = createAttendeeBundle(1);

  // In a "real" application, the Key should be defined in a constants file
  Intent intent = new Intent(ComplexObjectDemo.this, ComplexObjectReceiverDemo.class);
  intent.putExtra("test.complex.attendee", value);
   
  // Start the new activity
  startActivity(intent);
 }

 private void onCreateAttendee() {
  // Create a new AttendeeObject
  AttendeeObject attendee = new AttendeeObject();
  attendee.setName(mNameText.getText().toString());
  attendee.setAge(Integer.valueOf(mAgeText.getText().toString()));
  attendee.setIsPresent(mCurrentPresence);
  
  // You can then do whatever you want with this object
  // like adding it to an ArrayList or saving it to a database
  mAttendees.add(attendee);
  
  // Notify the user
  Toast.makeText(ComplexObjectDemo.this, "Attendee added!", Toast.LENGTH_SHORT).show();
 }

 public class MyOnItemSelectedListener implements OnItemSelectedListener {

     public void onItemSelected(AdapterView parent, View view, int pos, long id) {
      mCurrentPresence = (pos==0) ? true : false;      
     }

     public void onNothingSelected(AdapterView parent) { /*Do nothing.*/  }
 }
}
```

</details>


And here's the second activity:

<details>
<summary>Receiving Activity code</summary>

```java
package droidista.example;

import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;

public class ComplexObjectReceiverDemo extends Activity {
 
 @Override
 public void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_complex_demo_receiver);

  // Get the extras passed to this Activity 
  setUpViews(getIntent().getExtras());        
 }

 private void setUpViews(Bundle extras) {
  // Again, the key should be in a constants file!
  Bundle extraAttendee = extras.getBundle("test.complex.attendee");
  AttendeeObject attendee = new AttendeeObject();

  // Deconstruct the Bundle into the AttendeeObject
  attendee = attendee.unBundleAttendee(extraAttendee);

  // The AttendeeObject fields are now populated, so we set the texts
  ((TextView) findViewById(R.id.attendee_age_value)).setText(String.valueOf(attendee.getAge()));
  ((TextView) findViewById(R.id.attendee_name_value)).setText(attendee.getName());
  ((TextView) findViewById(R.id.attendee_presence_value)).setText(String.valueOf(attendee.isPresent()));
 }

}
```

</details>
