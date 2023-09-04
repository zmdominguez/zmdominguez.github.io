---
layout: post
title: Making ActionBarSherlock and ViewPagerIndicator play nice
date: '2012-08-14T16:50:00.000+10:00'
author: Zarah Dominguez
tags:
  - viewpagerindicator
  - actionbarsherlock
  - android
modified_time: '2012-12-28T02:27:44.331+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-1805460110318014182
blogger_orig_url: http://www.zdominguez.com/2012/08/making-actionbarsherlock-and.html
---

<b style="color: red;">EDIT (20121227): </b>I made a [new post](http://droidista.blogspot.com/2012/12/styling-tab-selectors-for.html) on changing the tab selector underline.  
<span style="color: red;"><b>EDIT (20121014)</b>:</span> I received feedback from the comments that rotating the device will cause the tab contents to revert to the default text. In essence, the adapter "loses" the contents of the tabs. I have corrected this in the example below as well as in github.  
<b><span style="color: red;">EDIT (20120905):</span></b> The full sample source code is now in [github](https://github.com/zmdominguez/vpi-abs-demo).

So [ViewPagerIndicator](http://viewpagerindicator.com/) cheerfully claims that it works with [ActionBarSherlock](http://actionbarsherlock.com/). I was trying to test this out today and I can't find a noob-friendly, step-by-step tutorial on how I can do it. I spent almost half a day frantically opening tab after tab in Google Chrome, but nothing seems to straight out say what I should do. So here it is, collated from several StackOverflow answers, blog posts, etc.

Get the ViewPagerIndicator (hereafter referred to as VPI) and ActionBarSherlock (hereafter referred to as ABS) libraries. Create your Android application in Eclipse and add those two projects as dependency library projects. If you do not know how to do that, see [here](http://developer.android.com/tools/projects/projects-eclipse.html#ReferencingLibraryProject).

First, create a theme. This is required by ABS, and will also help in VPI. What I did was to make the VPI theme inherit from the ABS theme so I don't lose any of the styles I use in other parts of the app. I just add or override items needed by the ViewPagerIndicator. In this example, I just changed the text size and color of the tab labels. You can go one step further and change the selected tab indicator (via state selectors), change how the selectors are drawn, etc.
```xml
<resources xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- This is our main ActionBarSherlock theme -->
    <style name="Theme.Styled" parent="Theme.Sherlock.Light.DarkActionBar">
        <item name="actionBarStyle">@style/Widget.Styled.ActionBar</item>
        <item name="android:actionBarStyle">@style/Widget.Styled.ActionBar</item>
    </style>
    
    <!-- This is our ViewPagerIndicator theme. We inherit from the ABS theme -->
    <style name="Theme.VPI" parent="Theme.Styled">
        <item name="vpiTabPageIndicatorStyle">@style/CustomTabPageIndicator</item>
    </style> 
    
    <!-- "Implementation" of our ABS custom theme -->
    <style name="Widget.Styled.ActionBar" parent="Widget.Sherlock.Light.ActionBar.Solid.Inverse">
        <item name="titleTextStyle">@style/TitleText</item>
        <item name="android:titleTextStyle">@style/TitleText</item>
    </style>
    
    <!-- "Implementation" of VPI theme. We just set the text size and color. -->
    <style name="CustomTabPageIndicator" parent="Widget.TabPageIndicator">
        <item name="android:textSize">50dip</item>
        <item name="android:textColor">#FFB33E3E</item>
    </style>
    
    <!-- More customizations for ABS -->
    <style name="TitleText" >
        <item name="android:textColor">@android:color/darker_gray</item>
        <item name="android:textSize">17dip</item>
    </style>
    
</resources>
```

Then we create the fragment(s) we need to populate the viewpager. My fragment is a simple layout with just a `TextView` indicating which tab is being shown.

```java
public class TestFragment extends SherlockFragment {
    private String mContent = "???";
    
    public static TestFragment newInstance(String text) {
        TestFragment fragment = new TestFragment();
        
        // Supply num input as an argument.
        Bundle args = new Bundle();
        args.putString(KEY_TAB_NUM, text);
        fragment.setArguments(args);
        
        return fragment;
    }
    
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        
        View view = inflater.inflate(R.layout.activity_main, null);
        String text = getString(R.string.tab_page_num) + mContent;
        ((TextView)view.findViewById(R.id.text)).setText(text);
        
        return view;
    }
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mContent =  getArguments() != null ? getArguments().getString(KEY_TAB_NUM) : "???";
    }
}
```

Then we create the activity that will hold everything together.
```java
public class VpiAbsTestActivity extends SherlockFragmentActivity {

    private static final String[] TAB_TITLES = new String[]{"This", "Is", "A", "ViewPager"};

    TestFragmentAdapter mAdapter;
    ViewPager mPager;
    PageIndicator mIndicator;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.simple_tabs);

        mAdapter = new TestFragmentAdapter(getSupportFragmentManager());

        mPager = (ViewPager) findViewById(R.id.pager);
        mPager.setAdapter(mAdapter);

        mIndicator = (TabPageIndicator) findViewById(R.id.indicator);
        mIndicator.setViewPager(mPager);
    }

    class TestFragmentAdapter extends FragmentPagerAdapter {
        private int mCount = TAB_TITLES.length;

        public TestFragmentAdapter(FragmentManager fm) {
            super(fm);
        }
        
        @Override
        public Fragment getItem ( int position){
            return TestFragment.newInstance(String.valueOf(position))
        }

        @Override
        public int getCount () {
            return mCount;
        }

        @Override
        public CharSequence getPageTitle ( int position){
            return TAB_TITLES[position];
        }
    }
}
```

And lastly, we apply the ViewPagerIndicator theme to our activity in the manifest:
```xml
<activity android:name="VpiAbsTestActivity" android:theme="@style/Theme.VPI" />
```

As it turns out, it _IS_ pretty straightforward!