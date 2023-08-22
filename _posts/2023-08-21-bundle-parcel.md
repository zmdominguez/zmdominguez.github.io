---
layout: post
title: "Bundling Things Nice and Pretty 💝"
tags:
    - android
    - parcelize
---

Of all the projects that I have worked on over the years, one thing they all have in common is the need to pass things around. Whether passing stuff to an `Activity` as `Intent` extras, a `Fragment` as arguments or its `onSaveInstanceState`, or even a `ViewModel`'s `SavedStateHandle`, the most common way to do it is through a [`Bundle`](https://developer.android.com/reference/android/os/Bundle).

An `Activity` can accept different types of data through the various `putExtra` methods, such as the usual `int`, `boolean`, `long`, etc., array versions of these types, or even `Parcelable`s. 

Let's take this data class, for example:
```kotlin
data class Person(
        val name: String,
        val rank: Int,
        // ...other fields omitted
)
```

Say we have another `Activity` called `DetailActivity` that needs the `Person`'s  `name` and the `rank`. We can pass these values individually via the relevant `putExtra` calls:
```kotlin
val detailIntent = Intent(this, DetailActivity::class.java)

detailIntent.putExtra(DetailActivity.EXTRA_KEY_NAME, person.name)
detailIntent.putExtra(DetailActivity.EXTRA_KEY_RANK, person.rank)
```

**Note**: In most circumstances, we would need to pass around minimal information such as an `ID`. However, there may be instances where we have to deal with more complex structures -- for example, when a user is applying filters to a list. For the purposes of this post, we will deal with multiple properties of a `data class`.  
{: .notice}

Here, I opted to define the `String` values for the keys as `const val`s in a `companion object` in `DetailActivity` so I don't have to type them over and over again:
```kotlin
class DetailActivity : AppCompatActivity() {
    // ...

    companion object {
        const val EXTRA_KEY_NAME = "dev.zarah.person.name"
        const val EXTRA_KEY_RANK = "dev.zarah.person.rank"
    }
}
```

And retrieve them in `DetailActivity`:
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    val name = intent.getStringExtra(EXTRA_KEY_NAME)
    val rank = intent.getIntExtra(EXTRA_KEY_RANK, 0)
}
```

This works, but IMHO it's not ideal. For one, we need to be extra careful that we are using the correct `get***Extra` call when retrieving the data. If we need to add another value to be passed, we need to change the code in a bunch of places: we  need to add a new key in the `companion object`, add another `putExtra` call in the originating `Activity`, and add another `get***Extra` call in the receiving `Activity`. If for some reason we need to change the type of any one of the extras, we should not forget to change the `get***Extra` call. The IDE cannot help us here, and we need to rely on our tests to catch any mismatch.

If we are working with `Fragment`s, the idea is similar but we need wrap the values together in a `Bundle` before sending them through as `arguments`. An `Activity` can also accept a `Bundle` as an extra, so we can use the [`bundleOf` convenience function](https://developer.android.com/reference/kotlin/androidx/core/os/package-summary#bundleOf(kotlin.Array)) to do the wrapping up:
```kotlin
val bundle = bundleOf(
        DetailFragment.EXTRA_KEY_NAME to person.name, 
        DetailFragment.EXTRA_KEY_RANK to person.rank, 
        )

// Passing into a `Fragment`
val fragment = DetailFragment()
fragment.arguments = bundle

// Passing into an `Activity`:
val detailIntent = Intent(this, DetailActivity::class.java)
detailIntent.putExtra(DetailActivity.EXTRA_KEY_AS_BUNDLE, bundle)
```

I think the `Bundle` approach is _slightly_ better for an `Activity` because it groups the information into one thing and if we want to refactor the `Activity` into a `Fragment` in the future, we already have a `Bundle` of stuff that we can use. However, we still need to remember to use the correct `get***` methods when retrieving values from the `Bundle`:
```kotlin
val bundleFromExtra = requireNotNull(intent.getBundleExtra(EXTRA_KEY_AS_BUNDLE))
val nameFromBundle = bundleFromExtra.getString(EXTRA_KEY_NAME)
val rankFromBundle = bundleFromExtra.getInt(EXTRA_KEY_RANK)
```

### `Parcel`-ing it up 🎁
The good news is that we can improve our implementation even more by using a [`Parcelable`](https://developer.android.com/reference/android/os/Parcelable), which both `Activity` and `Fragment` accept. I remember in my early days as an Android dev, I did not want to touch `Parcel`s with a ten-foot pole. But those days are gone and we now have the [`Parcelable` implementation generator](https://developer.android.com/kotlin/parcelize) that handles the boilerplate code required by `Parcelable`.

Going back to our example above, we can make a data class that would encapsulate the data we need to pass, annotate it with `@Parcelize`, and have it implement the `Parcelable` interface:
```kotlin
@Parcelize
data class DetailsExtras(
        val name: String,
        val rank: Int,
) : Parcelable
```
In some cases, there may not be a need to create a new `data class` just for extras or arguments. Annotating the `Person` class may work just as well if we need to pass everything that `data class` contains. For now, let us assume that there we do not want to pass through other information from `Person`, or perhaps we want to cobble together information from different models and thus need a new `data class`.

We can make a new instance of this `DetailsExtras` `data class` so we can pass it to an `Activity` or `Fragment`: 
```kotlin
val detailExtras = DetailActivity.Companion.DetailsExtras(
        name = person.name,
        rank = person.rank, 
        )
detailIntent.putExtra(DetailActivity.EXTRA_KEY_AS_PARCEL, detailExtras)
startActivity(detailIntent)
```
This is obviously personal preference, but when I need a `data class` for encapsulating extras I like putting in a `companion object` together with the key for the extra so that they live close together.

Retrieving the values is the same as before, except we only need to remember to retrieve a `Parcelable`:
```kotlin
// Pre-API33
val extras = requireNotNull(intent.getParcelableExtra<DetailsExtras>(EXTRA_KEY_AS_PARCEL))

// API33+
val extras = requireNotNull(intent.getParcelableExtra(EXTRA_KEY_AS_PARCEL, DetailsExtras::class.java))
val name = extras.name
val rank = extras.rank
```
With this approach, we do not have to worry about the types of `name` or `rank` because Kotlin is smart and can help us figure it out.

### Adding more stuff to our stuff 📝
What I really like about this approach is that it makes the code really predictable. There is no guessing which values may or may not be there, no guessing what types each of the values are, and any default values can be incorporated into the data class itself.

This also makes our implementation scalable and flexible -- we can even nest other `data class`es inside it if we so choose.

But perhaps the biggest benefit of all in my opinion is making the IDE do a lot of the thinking for us. Since we are using a `data class`, adding or removing a property (or changing its type) causes the IDE to flag all the places we need to update.

And if there's one thing I know for sure, it's that the earlier I let the IDE flag any errors before I need to rebuild my project, the better. 🏁 