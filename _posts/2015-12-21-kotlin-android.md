---
id: 585
title: Kotlin for Android Developers
date: 2015-12-21T01:00:27+00:00
author: Antonio Leiva
layout: post
guid: http://www.javaadvent.com/?p=585
permalink: /2015/12/kotlin-android.html
categories:
  - 2015
  - android
  - functional programming
  - kotlin
  - lambda
---
We Android Developers have a difficult situation regarding our language limitation. As you may know, **current Android development only support Java 6** (with some small improvements from Java 7), so we need to deal every day with a really old language that cuts our productivity and forces us to write tons of boilerplate and fragile code that it's difficult to read an maintain.

Hopefully, at the end of the day we're running over a Java Virtual Machine, so technically anything that can be run in a JVM is susceptible of being used to develop Android Apps. There are many languages that generate bytecode a JVM can execute, so some alternatives are starting to become popular these days, and **Kotlin is one of them**.

## What is Kotlin?

Kotlin is a language that runs on the JVM. It's being created by [Jetbrains](https://www.jetbrains.com/), the company behind powerful tools such as IntelliJ, one of the most famous IDEs for Java developers.

Kotlin is a really simple language. One of it's main goals is to provide a **powerful language with a simple and reduced syntax**. Some of it's features are:

- **It's lightweight**: this point is very important for Android. The library we need to add to our projects is as small as possible. In Android we have hard restrictions regarding method count, and Kotlin only adds around 6000 extra methods.
- **It's interoperable**: Kotlin is able to communicate with Java language seamlessly. This means we can use any existing Java library in our Kotlin code, so even though the language is young, we already have thousands of libraries we can work with. Besides, Kotlin code can also be used from Java code, which means we can create software that uses both languages. You can start writing new features in Kotlin and keep the rest of codebase in Java.
- **It's a strongly-typed language**: though you barely need to specify any types throughout the code, because the compiler is able to infer the type of variables or the return types of the functions in almost every situations. So you get the best of both worlds: a concise and safe language.
- **It's null safe**: One of the biggest problems of Java is null. You can't specify when a variable or parameter can be null, so lots of `NullPointerException` will happen, and they are really hard to detect while coding. Kotlin uses explicit nullity, which will force us check nulls when necessary.

Kotlin is currently in **version 1.0.0 Beta 3**, but can expect the final version very soon. It's quite ready for production anyway, there are already many companies successfully using it.

## Why Kotlin is great for Android?

Basically because all its features fit perfectly well in the Android ecosystem. The library is small enough to let us work without proguard during development. **It's size is equivalent to `support-v4` library**, and there are some other libraries we use in amost every projects that are even bigger.

Besides, Android Studio (the official Android IDE) is built over IntelliJ. This means our IDE have an excellent support to work with this language. **We can configure our project in seconds** and keep using the IDE as we are used to do. We can keep using Gradle and all the run and debug features the IDE provides. It's literally the same as writing the App in Java.

And obviously, thanks to its interoperability, we can **use the Android SDK without any problems from Kotlin code**. In fact, some parts of the SDK are even easier to use, because the interoperability is intelligent, and it for instance maps getters and setters to Kotlin properties, or let us write listeners as closures.

## How to start using Kotlin in Android

It's really easy. Just follow these steps:

- Download Kotlin plugin from the IDE plugins sections
- Create a Kotlin class in your module
- Use the action "Configure Kotlin in Project..."
- Enjoy

## Some features

Kotlin has a lot of awesome features I won't be able to explain here today. If you want to continue learning about it, you can [check my blog](http://antonioleiva.com/kotlin/) and [read my book](https://leanpub.com/kotlin-for-android-developers/). But today I'll explain some interesting stuff I hope it makes you want more.

### Null safety

As I mentioned before, Kotlin is null safe. If a type can be null we need to specify it by setting an `?` after the type. From that point, every time we want to use a variable that uses that type, we need to check nullity.

For instance, this code won't compile:

```
var artist: Artist? = null 
artist.print()
```

The second line will show an error, because the nullity wasn't checked. We could do something like this:

[scala]
if (artist != null) {
     artist.print() 
}
[/scala]

This shows another great Kotlin feature: Smart casting. If we've checked the type of a variable, we don't need to cast it inside the scope of that check. So we now can use `artist` as variable of type `Artist` inside the `if`. This works with any other check we may do (like after checking the instance type).

We have a simpler way to check nullity, by using `?` before calling a function of the object. And we can even provide an alternative by using the Elvis operator `?:`

[scala]
val name = artist?.name ?: &quot;&quot;
[/scala]

### Data classes
In Java, if we want to create a data class, or POJO class (a class that only saves some state), we'd need to create a class with lots fields, getters and setters, and probably a `toString` and an `equals` class:

[scala]
public class Artist {
    private long id;
    private String name;
    private String url;
    private String mbid;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getMbid() {
        return mbid;
    }

    public void setMbid(String mbid) {
        this.mbid = mbid;
    }

    @Override public String toString() {
        return &quot;Artist{&quot; +
                &quot;id=&quot; + id +
                &quot;, name='&quot; + name + '\'' +
                &quot;, url='&quot; + url + '\'' +
                &quot;, mbid='&quot; + mbid + '\'' +
                '}';
    }
}
[/scala]

In Kotlin, all the previous code can be substituted by this:

[scala]
data class Artist ( 
    var id: Long,
    var name: String,
    var url: String,
    var mbid: String)
[/scala]

Kotlin uses properties instead of fields. **A property is basically a field plus its getter and setter**. We can declare those properties directly in the constructor, that you can see is defined right after the name of the class, saving us some lines if we are not modifying the entry values.

The `data` modifier provides some extra features: a readable `toString()`, an `equals()` based on the properties defined in the constructor, a `copy` function, and even a set of `component` functions that let us split an object into variables. Something like this:

[scala]
val (id, name, url, mbid) = artist
[/scala]

### Interoperability

We have some great interoperability features that help a lot in Android. One of them is the **mapping of interfaces with a single method to a lambda**. So a click listener like this one:

[scala]
view.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View) {
        toast(&quot;Click&quot;) 
    }
 })
[/scala]

can be converted into this:

[scala]
view.setOnClickListener { toast(&quot;Click&quot;) }
[/scala]

Besides, **getters and setters are mapped automatically to properties**. This doesn't add any kind of overhead, because the bytecode will in fact just call to the original getters and setters. These are some examples:

[scala]
supportActionBar.title = title
textView.text = title
contactsList.adapter = ContactsAdapter()
[/scala]

### Lambdas

Lambdas will save tons of code, but the important thing is that it will let us do things that are impossible (or too verbose) without them. With them we can start thinking in a more functional way. A lambda is simply a way to specify a type that defines a function. We can for instance define a variable like this:

[scala]
val listener: (View) -&gt; Boolean
[/scala]

This is a variable that is able to declare a function that receives a view and returns a function. A closure is the way we have to define what the function will do:

[scala]
val listener = { view: View -&gt; view is TextView }
[/scala]

The previous function will get a `View` and return `true` if the view is an instance of `TextView`. Ad the compiler is able to infer the type, we don't need to specify it. We can be more explicit if we want by the way:

[scala]
val listener: (View) -&gt; Boolean = { view -&gt; view is TextView }
[/scala]

With lambdas, we can prevent the use of callback interfaces. We can just set the function we want to be called after and operation finishes:
	
[scala]
fun asyncOperation(value: Int, callback: (Boolean) -&gt; Unit) {
    ...
    callback(true) 
}

asyncOperation(5) { result -&gt; println(&quot;result: $result&quot;) }
[/scala]

But there is a simpler alternative, because if a function only has one parameter, we can use the reserved word `it`:

[scala]
asyncOperation(5) { println(&quot;result: $it&quot;) }
[/scala]

### Collections

Collections in Kotlin are really powerful. They are written over Java collections, so it means when we get a result from any Java library (or the Android SDK for instance), we still be able to use all the functions Kotlin provides.

The available collections we have are:

- Iterable
- Collection
- List
- Set
- Map

And we can apply a lot of operations to them. These are a few of them:

- filter
- sort
- map
- zip
- dropWhile
- first
- firstOrNull
- last
- lastOrNull
- fold
…

You may see the complete set of operations [in this article](http://antonioleiva.com/collection-operations-kotlin/). So a complex operation such as a filters, a sort and a transformation can be quite explicitly defined:

[scala]
parsedContacts
    .filter { it.name != null &amp;&amp; it.image != null }
    .sortedBy { it.name }
    .map { Contact(it.id, it.name!!, it.image!!) }
[/scala]

We can define new immutable lists in a simple way:

[scala]
val list = listOf(1, 2, 3, 4, 5)
[/scala]

Or if we want it to be mutable (we can add and remove items), we have a very nice way to access and modify the items, the same way we'd do with an array:

[scala]
mutableList[0] = 1
val first = mutableList[0]
[/scala]

And the same thing with maps:

[scala]
map[&quot;key&quot;] = 1
val value = map[&quot;key&quot;]
[/scala]

This is possible because we can [overload some basic operators](http://antonioleiva.com/operator-overloading-kotlin/) when implementing our own classes. 

### Extension functions

Extensions functions will let us **add extra behaviour to classes we can't modify**, because they belong to a library or an SDK for instance.

We could create an `inflate()` function for `ViewGroup` class:

[scala]
fun ViewGroup.inflate(layoutRes: Int): View {
    return LayoutInflater.from(context).inflate(layoutRes, this, false)
}
[/scala]

And from now on, we can just use it as any other method:

[scala]
val v = parent.inflate(R.layout.view_item)
[/scala]

Or even a `loadUrl` function to an `ImageView`. We can make use of Picasso library inside the function:

[scala]
fun ImageView.loadUrl(url: String) {
    Picasso.with(context).load(url).into(this)
}
[/scala]

All `ImageView`s can use this function now:

[scala]
contactImage.loadUrl(contact.imageUrl)
[/scala]

### Interface

**Interfaces in Kotlin can contain code**, which simulates a **simple multiple inheritance**. A class can be composed by the code of many classes, not just a parent. The interfaces can't, however, keep state. So if we define a property in an interface, the class that implements it must override that property and provide a value.

An example could be a `ToolbarManager` class that will deal with the `Toolbar`:

[scala]
interface ToolbarManager {
     val toolbar: Toolbar

     fun initToolbar() {
        toolbar.inflateMenu(R.menu.menu_main)
        toolbar.setOnMenuItemClickListener {
            when (it.itemId) {
                R.id.action_settings -&gt; App.instance.toast(&quot;Settings&quot;)
                else -&gt; App.instance.toast(&quot;Unknown option&quot;)
            }
            true
        }
    }
}
[/scala]

This interface can be used by all the activities or fragments that use a `Toolbar`:

[scala]
class MainActivity : AppCompatActivity(), ToolbarManager {
     override val toolbar by lazy { find&lt;Toolbar&gt;(R.id.toolbar) }

     override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        initToolbar()
        ...
    }
} 
[/scala]

### When expression

`When` is the alternative to `switch` in Java, but much more powerful. It can literally check anything. A simple example:

[scala]
val cost = when(x) {
    in 1..10 -&gt; &quot;cheap&quot;
    in 10..100 -&gt; &quot;regular&quot;
    in 100..1000 -&gt; &quot;expensive&quot;
    in specialValues -&gt; &quot;special value!&quot;
    else -&gt; &quot;not rated&quot;
}
[/scala]

We can check that a number is inside a range, or even inside a collection (`specialValues` is a list). But if we don't set the parameter to `when`, we can just check whatever we need. Something as crazy as this would be possible:

[scala]
val res = when {
    x in 1..10 -&gt; &quot;cheap&quot;
    s.contains(&quot;hello&quot;) -&gt; &quot;it's a welcome!&quot;
    v is ViewGroup -&gt; &quot;child count: ${v.getChildCount()}&quot;
    else -&gt; &quot;&quot;
}
[/scala]

### Kotlin Android Extensions

Another tool the Kotlin team provides for Android developers. It will be able to **read an XML and inject a set of properties** into an activity, fragment or view with the views inside the layout casted to its proper type.

If we have this layout:

[scala]
&lt;FrameLayout
    xmlns:android=&quot;...&quot;
    android:id=&quot;@+id/frameLayout&quot;
    android:orientation=&quot;vertical&quot;
    android:layout_width=&quot;match_parent&quot;
    android:layout_height=&quot;match_parent&quot;&gt; 
    &lt;TextView
        android:id=&quot;@+id/welcomeText&quot;
        android:layout_width=&quot;wrap_content&quot;
        android:layout_height=&quot;wrap_content&quot;/&gt;
 &lt;/FrameLayout&gt;
[/scala]

We just need to add this synthetic import:

[scala]
import kotlinx.android.synthetic.main.*
[/scala]

And from that moment, we can use the views in our `Activity`:

[scala]
override fun onCreate(savedInstanceState: Bundle?) {
    super&lt;BaseActivity&gt;.onCreate(savedInstanceState)
    setContentView(R.id.main)
    frameLayout.setVisibility(View.VISIBLE)
    welcomeText.setText(&quot;I´m a welcome text!!&quot;)
}
[/scala]

It's that simple.

### Anko

Anko is a library the Kotlin team is developing to simplify Android development. It's main goal is to **provide a DSL to declare views using Kotlin** code:

[scala]
verticalLayout {
    val name = editText()
    button(&quot;Say Hello&quot;) {
        onClick { toast(&quot;Hello, ${name.text}!&quot;) }
    }
}
[/scala]

But it includes **many other useful things**. For instance, a great way to navigate to other activities:

[scala]
startActivity&lt;DetailActivity&gt;(&quot;id&quot; to res.id, &quot;name&quot; to res.name)
[/scala]

It just receives a set of `Pair`s an adds them to a bundle when creating the intent to navigate to the activity (specified as the type of the function).

We also have direct access to system services:

[scala]
context.layoutInflater
context.notificationManager
context.sensorManager
context.vibrator
[/scala]

Or easy ways to create toasts and alerts:

[scala]
toast(R.string.message)
longToast(&quot;Wow, such a duration&quot;)

 alert(&quot;Yes /no Alert&quot;) {
    positiveButton(&quot;Yes&quot;) { submit() }
    negativeButton(&quot;No&quot;) {}
}.show()
[/scala]

And one I love, an simple **easy DSL to deal with asynchrony**:

[scala]
async {
    val result = longRequest()
    uiThread { bindForecast(result) }
}
[/scala]

It also provides a good set of tools to **work with SQLite and cursors**. The `ManagedSQLiteOpenHelper` provides a `use` method which will receive the database and can call directly to its functions:

[scala]
dbHelper.use {
    select(&quot;TABLE_NAME&quot;).where(&quot;_id = {id}&quot;, &quot;id&quot; to 20)
}
[/scala]

As you can see, it has a nice `select` DSL, but also a simple `create` function:

[scala]
db.createTable(&quot;TABLE_NAME&quot;, true,
        &quot;_id&quot; to INTEGER + PRIMARY_KEY,
        &quot;name&quot; to TEXT)
[/scala]

When you are dealing with a cursor, you can make use of some extension functions such as `parseList`, `parseOpt` or `parseClass`, that will help with parsing the result.

## Conclusion

As you can see, Kotlin simplifies Android development in many different points. It will boost your productivity and will let you solve usual problems in a very different and simpler way. 

My recommendation is that you at least try it and play a little with it. It's a really fun language and very easy to learn. If you think this language is for you, you may continue learning it by getting [Kotlin for Android Developers book](https://leanpub.com/kotlin-for-android-developers).