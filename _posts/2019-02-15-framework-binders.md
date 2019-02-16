---
layout: post
title: "Unwrapping Framework Binding Adapters"
tags:
- android
- data binding
---
For the past year or so, my team has been all-in with data binding. And if you know me at all, it obviously makes me one happy duck!

Over time the team has moved from seeing data binding as a Butterknife replacement to utilising more data binding features. In fact, we have built an arsenal of `@BindingAdapter`s already!

This is precisely what prompted me to look at what we have currently. And I noticed a curious thing. We have written some Binding Adapters that we did not _really_ need.

For example, we have this one for listening to the IME Action button:
```kotlin
@BindingAdapter("onEditorActionClicked")
fun onEditorActionClicked(editText: EditText, editorActionListener: EditTextEditorActionListener) {
    editText.setOnEditorActionListener { _, actionId, _ ->
        editorActionListener.onEditorActionClicked(actionId)
        false
    }
}
```
Thing is, we did not really need to write this custom `BindingAdapter` because one such thing exists in the framework.

It is understandable though that we missed this, because documentation on these things is a bit hard to come by. Even if we look at the framework source code, someone who is not too familiar with data binding might find it a bit hard to parse.

So let me try to help a little bit.

Whenever we want to implement a `BindingAdapter` -- especially if it is going to hook into an existing listener -- the first step should be to look at the Binding Adapters implemented in the framework. There's a whole bunch written for [all sorts of widgets](https://android.googlesource.com/platform/frameworks/data-binding/+/master/extensions/baseAdapters/src/main/java/android/databinding/adapters) (that is, [except for AndroidX SearchView](https://issuetracker.google.com/issues/122856766), but that's another story).

Since we want to work with the IME Action for an `EditText`, we should look at [`TextViewBindingAdapter`](https://android.googlesource.com/platform/frameworks/data-binding/+/master/extensions/baseAdapters/src/main/java/android/databinding/adapters/TextViewBindingAdapter.java).

At the top of the file, we see this annotation:
`@BindingMethod(type = TextView.class, attribute = "android:onEditorAction", method = "setOnEditorActionListener")`

The [documentation](https://developer.android.com/topic/libraries/data-binding/binding-adapters#specify-method) kind of glosses over what this means; but in a nutshell:
- `attribute` = when this attribute appears in a layout file
- `type` = then look for the implementation in this class
- `method` =  of a method with this name in the class defined in `type`

(Note: If you are interested in how the data binding library automagically sets the listeners and manages the interfaces, I highly suggest to checkout the generated binding file of your layout.)

Oooh notice how `setOnEditorActionListener` is the method we actually call in our own custom `BindingAdapter`! Looks like we are on to something!

Let's try to use this attribute then:
```xml
<EditText
    android:id="@+id/sample_edit_action"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:inputType="text"
    android:imeOptions="actionSend"
    android:onEditorAction="@{() -> handlers.onEditorActionClicked()}"/>
```
(Note: If you are not familiar with the lambda syntax I am using here, this is called a "listener binding" and you can read all about it [here](https://developer.android.com/topic/libraries/data-binding/expressions#event_handling).)

(Second note: Lint will tell you there is no such attribute. Lint lies!)

The next bit we have to do would be to tell our `handlers` what needs to happen when `onEditorAction` is called. For simplicity, let's say we want to show a `Toast` when the user clicks on the Send button.
```kotlin
fun onEditorActionClicked() : Boolean {
    Toast.makeText(this, "Send was clicked!", Toast.LENGTH_LONG).show()
    return false
}
```
The implementation is fairly straightforward, but one thing is important though: **_make sure the implementation has the same exact return value type as the interface_** (otherwise data binding [gets into a StackOverflowException](https://issuetracker.google.com/issues/123260053))!

Now what if we actually need the `KeyEvent` or the `ActionId`? Let's update our implementation to factor those in:
```kotlin
fun onEditorActionClicked(view: TextView, actionId: Int?, event: KeyEvent?) : Boolean {
    when(actionId) {
        EditorInfo.IME_ACTION_SEND -> {
            Toast.makeText(this, "Send was clicked!", Toast.LENGTH_LONG)
                    .show()
        }
    }
    return false
}
```

And let's update our layout file as well:
```xml
<EditText
    android:id="@+id/sample_edit_action"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:inputType="text"
    android:imeOptions="actionSend"
    android:onEditorAction="@{(v, id, event) -> handlers.onEditorActionClicked(v, id, event)}"/>
```
And we can finally delete our custom `BindingAdapter`! :tada:

:exclamation: Remember as well that data binding by default looks for methods prefixed with `set`. This means that if we can call a setter programmatically, we do not have to make a `BindingAdapter` for it. Some examples are `setEnabled()`, `setBackgroundColor()`, etc.

So in summary:  
- :no_good: If we want to call an existing setter, no need to make a custom `BindingAdapter`.
- :no_good: If we want to hook up an existing interface, no need to make a custom `BindingAdapter`.
- :ok_woman: When implementing a listener binding, the return values must match exactly
- :information_desk_person: When in doubt, explore existing framework binders

Have fun binding!
