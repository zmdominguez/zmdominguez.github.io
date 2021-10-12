---
title: "Hello! 👋"
layout: talks
permalink: /talks/
collection: speaking
entries_layout: grid
sort_by: date
sort_order: reverse
---
Android Developer. Sydney by way of Manila. Google Developer Expert. Baking noob.

{% include talks-section.html collection="speaking" blurb="I give talks about development in general and about Android in particular." %}

{% include talks-section.html collection="features" blurb="Some videos and/or featurettes that include me." %}

{% include talks-section.html collection="interviews" blurb="From time to time, I have a chat with other people and it ends up on the internet." %}


{% assign tag_name = "organising" %}
<section id="{{ tag_name | slugify | downcase }}" class="taxonomy-section">
  <h2 class="taxonomy-title">{{ tag_name }}</h2>

    <p>I also organise a number of events mainly in Sydney, Australia. Some of the major conferences I helped organise are:</p>
   
  {% capture organising %}
  [DevFest Sydney 2019](http://devfest.org.au/)  
  [DevFest Sydney 2018](http://2018.devfest.org.au/)  
  [DevFest Sydney 2017](http://2017.devfest.org.au/)  
  [DevFest Sydney 2016](http://2016.devfest.org.au/)  
  [DevFest Sydney 2015](http://2015.devfest.org.au/)  
  {% endcapture %}
  {{ organising | markdownify }}


    <a href="#page-title" class="back-to-top">{{ site.data.text[site.locale].back_to_top | default: 'Back to Top' }} &uarr;</a>
</section>

   