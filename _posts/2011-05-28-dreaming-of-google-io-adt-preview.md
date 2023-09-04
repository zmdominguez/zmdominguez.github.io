---
layout: post
title: 'Dreaming of Google I/O: ADT preview'
date: '2011-05-28T01:27:00.008+10:00'
author: Zarah Dominguez
tags:
- wishful thinking
- Google I/O
- android
modified_time: '2011-05-28T02:35:03.959+10:00'
thumbnail: https://img.youtube.com/vi/Oq05KqjXTvs/default.jpg
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-4102246993882403681
blogger_orig_url: http://www.zdominguez.com/2011/05/dreaming-of-google-io-adt-preview.html
---

It is my dream to one day attend Google I/O. But seeing as I'm from a Third World country where everyday is a practice in cost-cutting, it is very unlikely that I would fulfill that dream anytime soon. I haven't sat down and computed the actual cost, but thinking about it makes my head spin. Off the top of my head:
- US Visa application = P6500 (~USD150)
- Round trip plane fare ticket = P90,000 (~USD2000, if I'm lucky)-
- Google I/O ticket = P21,000 (~USD450, if my research is correct)

Then of course, there's food, transportation, accommodations, and pocket money. *sigh*

For now, I would have to content myself with watching videos from the sessions. Yesterday, I watched Android SDK tech lead's [Xavier Ducrohet](http://twitter.com/#!/droidxav)'s session on the Android Development Tools. I had a little interaction with Xavier on the Android developers Google group before, and it is refreshing to finally map a face to the name. I kinda feel bad too, because the few times that I talked to him in the groups, it is to complain about the problems I'm having whenever I update my ADT. Eclipse, the SDK and I have a love-hate relationship with each other. But that's another story.

Anyway, I was watching the session while having lunch which turned out to be a mistake. I was so awestruck by the new tools that I totally forgot to eat and my chicken got cold! I was gushing about the tools, but of course no one can relate as I am the only Android developer in our company. I was applauding and cheering by myself.  Oh well.

Needless to say, I am so impressed with the new version of ADT! (More exclamation marks should follow, but I'm trying to act mature) I used to hate making layout files. I am lightyears away from being artistic, I don't have a strong background in XML, and I simply find all the available features overwhelming. It's developer option paralysis, I would think. But the new ADT makes making layouts look like F-U-N! I love how it's sleeker, seemingly more user-friendly (I'd retain the qualifier until I get to try it out), and more feature-packed. 

Some of the reasons I'm already loving the new ADT:  
**1. Smarter extract to include**  
 The current ADT already has an extract to include feature, but the new version looks smarter. When you choose to extract a layout stub as an include, the ADT looks for the exact same layout in all your other layout files and replaces them with the include tag! Brilliant, I say!  

**2. XML editors: attribute auto-complete**  
Oh. My. God. I have been ranting about this for so long! There are simply too many attributes available for each type of widget I find it impossible to remember everything! And then of course I have to make sure that I am putting the correct value formats into those attributes! I admit that it terrified me when I was starting out in Android; having this feature saves the newbies from a lot of heartache.  

**3. Layout actions toolbar**  
Easier to set attributes without having to scroll down a lot then finding out you put in the wrong value and then having to scroll down again. Nifty!  

**4. Widget Previews in the palette**  
One time I was creating a layout file, I wanted to put an indeterminate progress bar but I want it to use the small progress bar style. But then I lost my list of native Android styles so I have to Google for the correct style that I want (`?android:attr/android:progressBarStyleSmall` is a little hard to remember).

Soon I can choose precisely what I want before dragging widgets onto the canvas! The previews plus the attribute autocomplete feature would work wonders, I would guess. And it is wonderful that the previews change with the selected theme!

**5. Support for more widgets**  
I used to think that I'm doing an `include` wrong, because the graphical editor doesn't show the layout I'm including. It turns out that I have to put it inside a `LinearLayout` just to have the editor render it properly. But it looks like those days are behind us now!

Also gone are the days when the ADT cannot render `TabHost`s! I'm doing my happy dance now!

**7. (Hopefully) easier drag-and-drop**  
Dragging-and-dropping in `RelativeLayout`s is a pain in the ass. The editor simply does not listen to me! But in the new ADT it looks like the new anchors are easier to understand than the current one, with squares all around.

I haven't tried dragging a view directly into the outline in the current ADT, but knowing that I would have it soon makes me happy!

**8. Custom views available in the GUI editor**  
And then Xavier's team showed us how much they love us by including custom views in the widget palette. I haven't really used that many custom views, but I guess a lot of developers who customize like crazy would love this feature.

---

That being said, I would like to thank Xavier and his team for making sure that us developers continue to love doing what we do by making our jobs a little easier. I would like to think that they take time to listen to our opinions, and they for sure make it easy for us to talk to them. I love how accessible they are, and how very supportive of us. Sending good vibes to you and your team, Xavier!

I just hope that having this new ADT would not make developers lazy. I wish that we still make sure we understand how the XML file works, that we still read the documentation for the widgets, that we simply do not rely on automation to do the work for us. For starters, make sure you know how to change an EditText to accept passwords, or phone numbers, or only letters without relying on the palette.

I admit that I screech like a true fan girl when Romain Guy or Xavier answer my tweets. *blush*

Here's the session's video. Gush with me!

<iframe width="480" height="295" src="http://www.youtube.com/embed/Oq05KqjXTvs?fs=1" frameborder="0" allowfullscreen=""></iframe></div></div></div>