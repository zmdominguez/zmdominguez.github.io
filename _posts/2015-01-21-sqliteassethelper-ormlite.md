---
layout: post
title: SQLiteAssetHelper + ORMLite
date: '2015-01-21T00:20:00.001+11:00'
author: Zarah Dominguez
tags:
  - database
  - sqlite
  - android
modified_time: '2015-01-21T00:23:46.595+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-6267826134161987884
blogger_orig_url: http://www.zdominguez.com/2015/01/sqliteassethelper-ormlite.html
---

I recently had cause to use Jeff Gilfelt's [SQLite Asset Helper library](https://github.com/jgilfelt/android-sqlite-asset-helper). For those unfamiliar, it is a library that can help with including a pre-populated SQLite database with your Android application. It is extremely convenient with unbundling a potentially huge database you would want to ship.

For the app I was working on, I also wanted to use [ORMLite](http://ormlite.com/sqlite_java_android_orm.shtml). This is another library that helps with persisting POJOs to SQLite databases. If you deal with a lot of persisted objects in your code, then this is probably something worth looking into.

I won't deal here with how ORMLite does its stuff, I'll leave it to you to go through the documentation. What I'll write about today is how to make these two libraries work together.

Right. Moving on. If you look at ORMLite's [sample code](https://github.com/j256/ormlite-examples/blob/master/android/HelloAndroid/src/com/example/helloandroid/DatabaseHelper.java), it mentions that your database helper class must extend `OrmLiteSqliteOpenHelper`. If you look at SQLite Asset Helper's [sample code](https://github.com/jgilfelt/android-sqlite-asset-helper/blob/master/samples/database-v1/src/main/java/com/example/MyDatabase.java), it mentions that your database helper class must extend `SQLiteAssetHelper`. What this means for us is that somehow, we need our database helper class to be able to talk to both of these libraries.

Since Android Studio is now the official IDE of choice, the [sample code for this post](https://github.com/zmdominguez/orm-sqliteassethelper) is now AS-compatible. Yay!

First, gradle:

```groovy
compile 'com.readystatesoftware.sqliteasset:sqliteassethelper:2.0.1'
compile 'com.j256.ormlite:ormlite-android:4.48'
```

When dealing with anything database, I tend to create my POJOs first. For this sample, I will be using the infamous Northwind database. For simplicity, this app will simply get the first entry from the `Employees` table and dump the contents into a `TextView`.

Since SQLite Asset Helper is the one who is in charge of our database creation and upgrade operations, we have to let it do the setup. Follow Jeff's example for that, BUT follow ORMLite's example for setting up ORMLite with [no helper](https://github.com/j256/ormlite-examples/tree/master/android/HelloAndroidNoHelper). At the end of it, you should have something similar [to this](https://github.com/zmdominguez/orm-sqliteassethelper/blob/master/app/src/main/java/droidista/blogspot/com/ormsqliteassethelper/orm/OrmDbHelper.java).

To actually use ORMLite, just do as you usually would:

```java
mOrmDbHelper = getHelper();

try {
    Dao<Employee, Integer> employeeDao = mOrmDbHelper.getEmployeeDao();
    // Try to get the first entry in the table
    Employee employee = employeeDao.queryBuilder().queryForFirst();
    if(employee == null) {
        textView.setText("No employees found!");
    } else {
        textView.setText(employee.toString());
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```

Here we just get the first entry we can from the table and dump. If all goes well when we run the app, we should see this in the logs:

```shell
01-20 23:02:39.203 &nbsp;12555-12555/droidista.blogspot.com.ormsqliteassethelper W/SQLiteAssetHelper﹕ copying database from assets...
01-20 23:02:39.226 &nbsp;12555-12555/droidista.blogspot.com.ormsqliteassethelper W/SQLiteAssetHelper﹕ database copy complete
01-20 23:02:39.289 &nbsp;12555-12555/droidista.blogspot.com.ormsqliteassethelper I/SQLiteAssetHelper﹕ successfully opened database northwind.db
```

And we should see Nancy's details displayed on screen in all it's raw `.toString()` glory.