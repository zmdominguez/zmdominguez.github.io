---
layout: post
title: "Effectively Wrangling Together a Bunch of Views"
tags:
- android
- data binding
- constraint layout
---
One bit of task that I find myself doing over and over again is managing a bunch of `View`s and their visibility. In the olden days *&lt;insert old person handwave&gt;*, before there was `ConstraintLayout`, I have written my fair share of `container_`s to make this task manageable. Say we have to do something like this:

<p style="text-align: center">
    <a href="https://imgur.com/aMHWlb2"><img src="https://i.imgur.com/aMHWlb2.gif"  width="320" /></a><br />
<small>Now you see me, now you don't!</small></p>  

If you asked me two years ago to implement the layout above, I would have:
- wrapped the divider, the section title, and that extra text in a `LinearLayout`
- added an `OnCheckedChangeListener` to the `CheckBox`
- managed the `LinearLayout`'s visibility in the listener implementation

Or in code:
```java
@BindView(R.id.checkbox) CheckBox mCheckbox;                                                   
@BindView(R.id.container) ViewGroup mContainer;                                                
                                                                                               
@Override                                                                                      
public void onCreate(@Nullable Bundle savedInstanceState) {                                    
    super.onCreate(savedInstanceState);                                                        
    setContentView(R.layout.activity_constraint);                                              
    ButterKnife.bind(this);                                                                    
    mCheckbox.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {        
        @Override
        public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) { 
            if (isChecked) {                                                                   
                mContainer.setVisibility(View.VISIBLE);                                        
            } else {                                                                           
                mContainer.setVisibility(View.GONE);                                           
            }                                                                                  
        }                                                                                      
    });                                                                                        
}
```
(Here is the `Activity` code [in full](https://github.com/zmdominguez/sdk_sandbox/blob/master/app/src/main/java/com/zdominguez/sdksandbox/ConstraintLayoutDemo.java), as well as [the layout file](https://github.com/zmdominguez/sdk_sandbox/blob/master/app/src/main/res/layout/activity_constraint_compat.xml)).

It works, but it's a whole lot of things to write!

Let's try to harness the power of the tools we currently have at our disposal. First off, here's the skeletal `ConstraintLayout`-ified version of our existing layout file (click [here for the full version](https://github.com/zmdominguez/sdk_sandbox/blob/master/app/src/main/res/layout/activity_constraint.xml)):
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
    </data>
    <android.support.constraint.ConstraintLayout>

        <ImageView android:id="@+id/image_1" />

        <ImageView android:id="@+id/image_2" />

        <TextView android:id="@+id/description" />

        <CheckBox android:id="@+id/checkbox" />

        <View android:id="@+id/divider" />

        <TextView android:id="@+id/section_description" />

        <TextView android:id="@+id/section_title" />

        <Button android:id="@+id/button_2"/>
    </android.support.constraint.ConstraintLayout>
</layout>

```
Immediately we see that the view hierarchy is so much flatter, and there are no unnecessary nestings at all! But then, the next question arises: doesn't this mean that when we have to deal with managing the visibility of the required widgets we could either end up writing *more* code or introduce some level of nesting?

Well, not quite! We can use a `ConstraintLayout` feature called [`Group`](https://developer.android.com/reference/android/support/constraint/Group) that would make managing our desired behaviour much much easier. A `Group` allows us to reference a comma-separated list of widget IDs and directly manage their visibility -- which is exactly what we want to achieve!

Let's go ahead and make the `Group`:

```xml
<android.support.constraint.Group
    android:id="@+id/group"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:constraint_referenced_ids="divider,section_description,section_title"/>
```
And now all that's left is actually hiding or showing it. For this demo, I opted to use the same technique as in my [previous post](https://zdominguez.com/2018/10/13/databinding-id-ref.html) and use a data binding expression:
```xml
android:visibility="@{checkbox.checked ? View.VISIBLE : View.GONE}"
```
We can now simplify our `Activity` code to become:
```kotlin
public override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    DataBindingUtil.setContentView<ActivityConstraintBinding>(this, R.layout.activity_constraint)
}
```
Fun!
