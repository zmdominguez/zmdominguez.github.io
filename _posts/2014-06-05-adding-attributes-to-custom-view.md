---
layout: post
title: Adding attributes to a custom view
date: '2014-06-05T17:24:00.001+10:00'
author: Zarah Dominguez
tags:
  - EditText
  - android
  - layout
modified_time: '2014-06-06T03:02:21.812+10:00'
thumbnail: http://3.bp.blogspot.com/-5U8Vj7Dea5E/U5AZzCIJ4BI/AAAAAAAABJc/BjeIfMRrNpA/s72-c/Screen+Shot+2014-06-05+at+3.18.02+PM.png
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-6564421430248280506
blogger_orig_url: http://www.zdominguez.com/2014/06/adding-attributes-to-custom-view.html
---

There are times when using the default Android Views just doesn't cut it and you need to create your own version of a View. So how exactly do you do that? It's as simple as subclassing the View! But what if you want to add customizable attributes? Here's how.

Let's say I am creating a form-filling application and I want some of the `EditText`s in the form to be required. However, I am _so_ tired of having to implement the error checking for each and every one of those fields. What I will do is create my own `EditText` that will do the validation for me if that particular field is required. Let's do it.

Step 1: Create your custom view and create fields for the attributes you want. In this case, I want an `EditText` that will show the default `EditText` error display if the user has not put in any value. 

```java
public class RequiredEditText extends EditText {
    private boolean mRequired
    private String mErrorMessage;
    
    public RequiredEditText(Context context) {
        super(context);
    }

    /**
     * Set this EditText's requirement validation. The error message
     * will be set to <code>null</code> by default if not provided.
     * 
     * @param required
     * @param errorMessage (optional)
     */
    public void setRequired(boolean required, String errorMessage) {
        this.mRequired = required;
        this.mErrorMessage = errorMessage;
        
        invalidate();
        requestLayout();
    }
    
    public void setRequired(boolean required) {
        setRequired(required, null);
    }

    /** Lets you know if this field is set as required or not
     * @return
     */
    public boolean isRequiredField() {
        return mRequired;
    }
}
```

Step 2: If a field is required, we want the default error message to appear (with our own error message, of course). If the user fills in the `EditText`, we want the error to disappear.  

```java
public class RequiredEditText extends EditText {
    private boolean mRequired;
    private String mErrorMessage;
    
    public RequiredEditText(Context context) {
        super(context);
    }
    
    
    /** 
     * Set this EditText's requirement validation. The error message 
     * will be set to <code>null</code> by default if not provided.
     * 
     * @param required
     * @param errorMessage (optional)
     */ 
    public void setRequired(boolean required, String errorMessage) {

        this.mRequired = required;
        this.mErrorMessage = errorMessage;
        manageRequiredField(required);
        
        invalidate();
        requestLayout();
    }
    
    public void setRequired(boolean required) {
        setRequired(required, null);
    }

    private void manageRequiredField(boolean required) {
        // If we are required, set the listeners
        if(required) {
            setOnFocusChangeListener(mFocusChangeListener);
            addTextChangedListener(mTextWatcher);
        } else {
            // In case there is an error message already, clear it
            setError(null);

            // Remove the listeners
            setOnFocusChangeListener(null);
            removeTextChangedListener(mTextWatcher);
        }
    }

    /**
     * Lets you know if this field is set as required or not
     * @return
     */
    public boolean isRequiredField() {
        return mRequired;
    }

    OnFocusChangeListener mFocusChangeListener = new OnFocusChangeListener() {

        @Override
        public void onFocusChange(View v, boolean hasFocus) {
            // If the focus was removed from the field and it IS required,
            // check if the user has put in something
            if(!hasFocus && mRequired){
                isRequiredFieldFilled();
            }
        }
    };

    TextWatcher mTextWatcher = new TextWatcher() {

        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {
            // Once the user types in something, remove the error
            setError(null);
        }

        @Override
        public void beforeTextChanged(CharSequence s, int start, int count, int after) { /* do nothing */ }

        @Override
        public void afterTextChanged(Editable s) { /* do nothing */ }
    };

    private boolean isRequiredFieldFilled() {
        // If the EditText is empty, show the error message
        if(TextUtils.isEmpty(getText().toString().trim())){
            showRequiredErrorDrawable();
            return false;
        }
        return true;
    }

    private void showRequiredErrorDrawable() {
        setError(mErrorMessage);
    }
}
```

Step 3: Now it's time to add the fields to the Layout Editor. Create an `attrs.xml` file in `/res` if there isn't one already. Declare a `styleable` and include the attributes you want to appear in the Layout Editor.
```xml
<resources>
    <declare-styleable name="RequiredEditText">
        <attr format="boolean" name="required" />
        <attr format="string" name="errorMessage" />
    </declare-styleable>
</resources>
```

The name of the `styleable` does not need to match your custom view class name, but doing so makes it more readable and maintainable.

Step 4: Go back to your custom view implementation and add constructors that take in an `AttributeSet`.
```java
public RequiredEditText(Context context, AttributeSet attrs, int defStyle) {
   super(context, attrs, defStyle);
   init(attrs);
}

public RequiredEditText(Context context, AttributeSet attrs) {
   super(context, attrs);
   init(attrs);
}

private void init(AttributeSet attrs) { 
   TypedArray a=getContext().obtainStyledAttributes(
      attrs,
      R.styleable.RequiredEditText);

   try {
      setRequired(a.getBoolean(R.styleable.RequiredEditText_required, false), 
            a.getString(R.styleable.RequiredEditText_errorMessage));
   } finally {
      //Don't forget this, we need to recycle
      a.recycle();
   }
}

```

Step 5: Go to the Layout Editor and look for "Custom & Library Views" in the Palette (you may have to click on "Refresh" several times before your custom view appears in the list). Add the custom view to your layout and check out the properties panel!

<a href="http://3.bp.blogspot.com/-5U8Vj7Dea5E/U5AZzCIJ4BI/AAAAAAAABJc/BjeIfMRrNpA/s1600/Screen+Shot+2014-06-05+at+3.18.02+PM.png" imageanchor="1"><img border="0" src="http://3.bp.blogspot.com/-5U8Vj7Dea5E/U5AZzCIJ4BI/AAAAAAAABJc/BjeIfMRrNpA/s320/Screen+Shot+2014-06-05+at+3.18.02+PM.png" /></a>

As always, the code is in [GitHub](https://github.com/zmdominguez/custom-view).