---
layout: post
title: MongoDB and Authentication
date: '2011-12-20T13:49:00.000+11:00'
author: Zarah Dominguez
tags:
  - authentication
  - mongodb
modified_time: '2012-02-13T14:37:59.609+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-1482053482459432531
blogger_orig_url: http://www.zdominguez.com/2011/12/mongodb-and-authentication.html
---

By default, MongoDB allows access to the database without authentication. Adding a user with a username/password is easy, but authenticating might be a bit tricky since the [official documentation](http://www.mongodb.org/display/DOCS/Security+and+Authentication) does not say the command directly. 

First, we add an admin account. Navigate to the MongoDB directory on your machine then start the database. 
```shell
$ ./mongo
> use admin
> db.addUser(adminuser, adminpassword)
```

Switch to the database of your choice and add users to it. 
```shell
> use foo
> db.addUser(myuser, userpassword)
```

This adds a user `myuser` that has read and write access to the database. If we want a user with read-only access, set the third parameter for `addUser()`. 

```shell
> db.addUser(guest, guestpassword, true)
```

You can check for users with access to a particular database like thus:
```shell
> db.system.users.find().pretty()
{
        "_id" : ObjectId("4ee9863d954eb7168e07089d"),
        "user" : "zarah",
        "readOnly" : false,
        "pwd" : "70581bfb1e32e2286df11fe119addc7a"
}
{
        "_id" : ObjectId("4ee98658954eb7168e07089e"),
        "user" : "guest",
        "readOnly" : true,
        "pwd" : "88558f1ece63fa0b528012b9840bd9de"
}
```

Now stop the MongoDB server and restart it with authentication enabled. 
```shell
$ ./mongod --auth
> mongo foo -u myuser -p userpassword<
```
where `foo` is the database that `myuser` has access to. 
You can now read and write into database `foo`. Notice however that querying for databases would result to an error:
```shell
> show dbs
Mon Dec 19 17:21:20 uncaught exception: listDatabases failed:{ "errmsg" : "need to login", "ok" : 0 }
```

Exit MongoDB and login again, this time using the read-only account. If we try inserting a document, an error should appear:
```shell
> db.foo.insert({"title","MongoDB Authentication Test"})
unauthorized
```

The read-only account can query for collections and use `find()` and its variations. It can't, however, query for databases.