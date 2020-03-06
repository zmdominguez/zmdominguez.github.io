---
layout: post
title: "//TODO Live Templates"
tags:
    - android studio
---
Throughout my career, I have worked in projects of all sizes. I have taken part in greenfield projects and some that are a few years old. One of the lessons I have learned over the years is that no one ever goes back to fix the `TODO`s.

In our current project, we are trying to mitigate the unchecked growth of this list (we have some from 2015 :sob:). One of the solutions we are trying to explore is to create labeled `TODO`s:

```kotlin
// TODO-Zarah (06 Mar 2020): Some comments go here
```

This means that when leaving a `TODO`, devs would have to leave their name and the date in the comment. We then do periodic checks (usually before a release) to make sure that we are actively going back and **actually** fixing the issues.

Now typing the same thing over and over is indeed very annoying,
<center>
    <div class="tenor-gif-embed" data-postid="10243842" data-share-method="host" data-width="50%" data-aspect-ratio="1.0"><a href="https://tenor.com/view/nobody-got-time-for-that-gif-10243842">Nobody Got Time For That AIntnobody Got Time For That GIF</a> from <a href="https://tenor.com/search/nobodygottimeforthat-gifs">Nobodygottimeforthat GIFs</a></div><script type="text/javascript" async src="https://tenor.com/embed.js"></script>
</center>

To help alleviate the pain, we decided to employ [parameterised live templates](https://www.jetbrains.com/help/idea/using-live-templates.html). There are a whole bunch of these templates (`Preferences` > `Editor` > `Live Templates`) available in Android Studio, like the one for making a `Toast`:
<center>
    <a href="https://imgur.com/BrrYoEp"><img src="https://i.imgur.com/BrrYoEp.gif" title="source: imgur.com" /></a>
    <br /> <small>Never forget .show() again</small>
</center>

To create our template, open `Preferences` > `Editor` > `Live Templates` then click on the `+` sign on the right of the panel. I chose to create a `Template Group` to contain our custom templates, but it's also fine to directly add a new item to any of the existing groups.

<center>
    <a href="https://imgur.com/s7QvcT4"><img src="https://i.imgur.com/s7QvcT4.png" title="source: imgur.com" /></a>
    <br /> <small>Template creation menu</small>
</center>

### Live Template Anatomy

Android Studio will ask for some information when creating templates:

<center>
    <a href="https://imgur.com/yHpXt55"><img src="https://i.imgur.com/yHpXt55.png" title="source: imgur.com" /></a>
    <br /> <small>Live template anatomy</small>
</center>

**Abbreviation**: What the user should type to use a template  
**Description**: Short text to appear in the context menu  
**Template text**: The actual template, including variables  
**Context**: Gives Android Studio hints on when it should suggest the template (choosing Java and Kotlin is usually sufficient)

### Variables

[Variables](https://www.jetbrains.com/help/idea/template-variables.html) are either pre-defined values or input fields. In our case, we have several of these:
- person the `TODO` is assigned to
- the date the `TODO` was left
- the actual comment for the `TODO` (optional, can be left out of the template)

To auto-populate the variable values, click on the `Edit Variables` button and define the expressions to use. There are a lot of [pre-defined functions](https://www.jetbrains.com/help/idea/template-variables.html#predefined_functions) we can leverage.

<center>
    <a href="https://imgur.com/fkoPOAD"><img src="https://i.imgur.com/fkoPOAD.png" title="source: imgur.com" /></a>
    <br /> <small>Variable editing</small>
</center>

On my work computer, the `user()` function gives back my work-imposed ID and it's not pretty. To use a better (i.e., my *actual* name) value without having to type it over and over, we can override Android Studio's custom properties (`Help` > `Edit custom properties`) and add the value:
```
-Duser.name=Zarah
```

And here's our brand new template in action:
<center>
    <a href="https://imgur.com/LjaiHU5"><img src="https://i.imgur.com/LjaiHU5.gif" title="source: imgur.com" /></a>
    <br /> <small>Custom template in action</small>
</center>

### Filtering `TODO`s
At the start of this post I mentioned that we do periodic checks on our `TODO`s. When I look at our `TODO` panel (`View` > `Tool Windows` > `TODO`), there are hundreds of them across *so many* files. Before a release, we review all the templated ones so we can follow up with the devs who left the comments.

To make this task easier, we created a `TODO` filter:
<center>
    <a href="https://imgur.com/QtOh8qv"><img src="https://i.imgur.com/QtOh8qv.png" title="source: imgur.com" /></a>
    <br /> <small>TODO filters</small>
</center>

Unfortunately the results of this search is not exportable, so if we want to generate a report we need to run code analysis (`Analyze` > `Run inspection by name`) and choose `TODO`. *This* can be exported to either HTML or XML which makes it easier to share with the team.
