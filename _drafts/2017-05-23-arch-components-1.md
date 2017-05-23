---
layout: post
title: "There is Always Room for Improvement"
tags:
- io17
- android
- arch
---
During the beginning of my Android career, one of the things I had to do was to save some data to _somewhere_. I found out that for my purposes, I had to use an SQLite database. Reading through the [docs](https://developer.android.com/training/basics/data-storage/databases.html), I was afraid. Petrified, even.

What do you mean I have to write `Create TABLE` scripts? I have to do what now to upgrade a table? Do not forget that _required_ `_ID` column! What do you mean I have to deal with `Cursor`s? Oh that means I have to define constants so that I don't typo table and column names all the time. Where is this cursor that I forgot to close? I have to remember column positions? What?

There are just so many pitfalls.

[Room](https://developer.android.com/topic/libraries/architecture/room.html), the new architecture component [introduced during IO this year](https://youtu.be/MfHsPGQ6bgE), aims to help solve most -- if not all -- of these problems.

`Room` abstracts away the nasty SQL-related boilerplate code we need to do and instead gives us an easy, fluent way of declaring databases, tables, and their structures.

For today, we will aim to duplicate (and eventually shift) Topeka's existing table of available quiz categories to `Room`. [Topeka](https://github.com/googlesamples/android-topeka) is a sample project from Google's DevRel team that aims to demonstrate material design in action.

Getting started with room is fairly easy, with the components being available in the recently-released Maven repository (!!!!!!):
```
maven { url 'https://maven.google.com' }
```

And in your project's build file:
```
ext {
    roomVersion = "1.0.0-alpha1"
}

dependencies {
    compile "android.arch.persistence.room:runtime:${roomVersion}"
    annotationProcessor "android.arch.persistence.room:compiler:${roomVersion}"
}
```

We will now go through each of the basic building blocks of `Room` one-by-one.

First off, the _**database**_. For now, we will focus on converting the `Category` table in the existing database. Let's go ahead and create the abstract class to represent it:
```java
@Database(entities = {Category.class}, version = 1)
public abstract class TopekaRoom extends RoomDatabase {
}
```

Here, we tell `Room` what tables we have via the `entities` property of the `@Database` annotation, give it our version number and that's it. 

Next, we need to tell `Room` a little more detail about this _**entity**_. Topeka already has an existing class called [`Category`](https://github.com/googlesamples/android-topeka/blob/master/app/src/main/java/com/google/samples/apps/topeka/model/Category.java) which sort of maps to the current columns in its database. The beauty of `Room` is that we can re-use this model and make our new table work side-by-side with the old one.

To convert this model into a table, we simply annotate the class:
```java
@Entity(tableName = "category")
public class Category implements Parcelable {
    // ...
}
```

We also need to tell `Room` which of the fields in our model we want stored in our database. [`CategoryTable`](https://github.com/googlesamples/android-topeka/blob/master/app/src/main/java/com/google/samples/apps/topeka/persistence/CategoryTable.java) gives us an idea:
```
String CREATE = "CREATE TABLE " + NAME + " ("
            + COLUMN_ID + " TEXT PRIMARY KEY, "
            + COLUMN_NAME + " TEXT NOT NULL, "
            + COLUMN_THEME + " TEXT NOT NULL, "
            + COLUMN_SOLVED + " TEXT NOT NULL, "
            + COLUMN_SCORES + " TEXT);";
```

Looks like at a minumum, we need to add the `@PrimaryKey` annotation to the ID field:
```java
private final String mName;

@PrimaryKey
private final String mId;

private final Theme mTheme;
private final int[] mScores;
private List<Quiz> mQuizzes;
private boolean mSolved;
```

And for the final component of `Room`, we need to make an interface that indicates what operations we want to perform on our entities.
```java
@Dao
public interface CategoryDao {

    @Query("SELECT * FROM category")
    List<Category> getAll();

    @Insert
    void insertCategory(Category category);

    @Query("DELETE FROM category")
    void deleteAll();
}
```
If you use Retrofit, this pattern may seem familiar.

We also need to tell our `@Database`-annotated class of our new DAO:
```java
@Database(entities = {Category.class}, version = 1)
public abstract class TopekaRoom extends RoomDatabase {
    public abstract CategoryDao categoryDao();
}
```
`Room` will generate the implementation of this DAO for us at compile time.

Now that we have set everything up, it should just work right? Not quite. Running the app spits out a bunch of errors.
<p style="text-align: center"><a href="Room needs help"><img src="{{ site.baseurl }}/assets/cannot_figure_out_convert.png" ></a></p>

Double-clicking the error brings us to the problematic field. `Room` cannot figure out how we want `List<Quizzes>` to be stored. We really don't want it in this table anyway, so we can safely tell `Room` to ignore it:
```java
@Ignore
private List<Quiz> mQuizzes;
```

The next error we have to deal with is that our `Category` model has a bunch of constructors present and `Room` cannot figure out which one we want to use.
<p style="text-align: center"><a href="Room needs more help"><img src="{{ site.baseurl }}/assets/cannot_figure_constructor.png" ></a></p>
We can fix this by making the fields we want in our entity to be public, or by making the required setters for our fields (which have been declared final), or by making a suitable constructor that `Room` can use. A "suitable constructor" means both types and names match the fields of our `Entity`.

```java
public Category(@NonNull String name, @NonNull String id, @NonNull Theme theme, boolean solved, int[] scores) {
        mName = name;
        mId = id;
        mTheme = theme;
        mScores = scores;
        mSolved = solved;
    }
```

The last bit of error we need to fix has something to do with conversions. According to the docs:
>Room provides built-in support for primitives and their boxed alternatives. However, you sometimes use a custom data type whose value you would like to store in the database in a single column.  

In our case, we want to store the `Theme` but `Room` does not know how to do it. We can solve this by making a `Converter` for the non-primitive types we have:
```java
public class Converters {
    @TypeConverter
    public static String themeToString(Theme theme) {
        return theme.name();
    }

    @TypeConverter
    public static Theme stringToTheme(String themeName) {
        return Theme.valueOf(themeName);
    }
}
```
And tell `Room` about it:
```java
@Database(entities = {Category.class}, version = 1)
@TypeConverters({Converters.class})
public abstract class TopekaRoom extends RoomDatabase {
    public abstract CategoryDao categoryDao();
}
```
We have almost reached our goal for today! The only thing remaining is to hook up our `Room` to what currently powers our existing database. One thing to remember here is that `Room` does not allow you to access your database from the main thread and fails with an `IllegalStateException` -- for now, we can wrap our code in an `AsyncTask` to see if we have set up everything correctly.

For Topeka, we want to reset the database into an empty state when the user logs out. We can therefore make a method that fills up our `Room` with the initial state of the database.
```java
TopekaRoom.getARoom(context).categoryDao().insertCategory(category);
```

If we are inserting a huge chunk of data, `Room` supports transactions like traditional SQL.
```java
room.beginTransaction();
try {
    fillCategoriesAndQuizzes(room, context);
    room.setTransactionSuccessful();
} finally {
    room.endTransaction();
}
```

Running our app now shows both of our databases:
<p style="text-align: center"><a href="We built a Room"><img src="{{ site.baseurl }}/assets/topeka_dbs.png" ></a></p>

Success!

You can view the full source code for this post in my [Topeka fork](https://github.com/zmdominguez/android-topeka). 



