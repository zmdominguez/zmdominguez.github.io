---
layout: post
title: Adding Preferences On-The-Fly
date: '2012-05-22T17:49:00.001+10:00'
author: Zarah Dominguez
tags:
  - shared preferences
  - android
modified_time: '2012-05-22T17:49:16.569+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-6535886254769330142
blogger_orig_url: http://www.zdominguez.com/2012/05/adding-preferences-on-fly.html
---

(Make appropriate whooshing sounds)

Today, I will show you how to add preferences to your `SharedPreferences` (confused yet?) programatically. In an app that I am doing, I wanted to present a [ListPreference](http://developer.android.com/reference/android/preference/ListPreference.html) to the user from the SharedPreferences page. However, the values contained in this list is not known at compile time. (Think Facebook groups that differ per user, or Twitter lists, or things like such, and I want to set a default option.)

Anyway, here's what I did to make this work. I added a preference in my `PreferenceScreen`, a sort of placeholder, if you will. Here is my `ListPreference`:
```xml
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android" >
    
    <PreferenceCategory android:title="@string/pref_settings_header">
        <ListPreference android:key="@string/pref_key_default_group" android:title="@string/pref_default_group"/>
    </PreferenceCategory>

</PreferenceScreen>
```

And then in the class that handles the SharedPreferences, I get the values that I want displayed and plug them in to the placeholder preference. I will not bother to explain here, just heavily commenting the code:
```java
public class UserPreferences extends PreferenceActivity {

    private static final String LOG_TAG = "UserPreferences";

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Load the preferences from an XML resource
        addPreferencesFromResource(R.xml.my_apps_prefs);

        // Manage the "Default Group" preference
        addDefaultGroupPrefs();
    }


    private void addDefaultGroupPrefs() {
        // Get our "placeholder" preference. I prefer to use strings instead of hard-coding the preference key.
        // The value passed into findPreference is the value in the android:key parameter in the preference XML file.
        ListPreference groupsListPrefs = (ListPreference) findPreference(getString(R.string.pref_key_default_group));

        // Get your groups, this may come from a utility class or wherever
        // In this example, I have a custom object that has a "name" and an "id"
        List<CustomGroup> myGroups = getMyGroups();
        if (myGroups == null) {
            // Disable the preference if we don't have a list
            groupsListPrefs.setEnabled(false);
        } else {
            // Enable the preference if we have a list
            groupsListPrefs.setEnabled(true);

            // We split the list into names and IDs
            // We show the names in the list, and the values as the entries
            List<String> groupNames = new ArrayList<String>();
            List<String> groupIds = new ArrayList<String>();
            for (CustomGroup group : myGroups) {
                groupIds.add(group.getId());
                groupNames.add(group.getName());

                // I guess you can add directly to a String[]
                // just make sure the order is preserved (name1=id1, name2=id2, etc)
            }

            // These should be user-readable strings
            groupsListPrefs.setEntries(groupNames.toArray(new String[groupNames.size()]));

            // These are the actual values to be saved in the XML file
            groupsListPrefs.setEntryValues(groupIds.toArray(new String[groupIds.size()]));
        }
    }

    private List<customgroup> getMyGroups() {
        // do work here, query server or db or whatever
        // return the list of groups
    }
}
```

And we're done. :)

