---
layout: post
title: "GSoC Blog week 6"
description: ""
category: 
tags: [gsoc]
---
{% include JB/setup %}

###Stanbol Related stuff
I had issues building the latest stanbol head because of obsolete cache in
`/sling`. After discussing it with the stanbol team, I succeeded to build it and
to run it. In this head, major changes have occured :

Store has been replaced by ContentHub, which works the same way as Store did
New Entry Points have been added : `/ontonet`, `/reasoners` and `/rules`
I will have to update the Store class in `stanbol.client` to match the new stanbol
API, and implement new Entry Points.

###Plone Related stuff
I pushed the changes made on `stanbol.client`, `stanbol.plone` and `stanbol-buildout`
on three forks on github :

- <http://github.com/ymazzer/stanbol.client>
- <http://github.com/ymazzer/stanbol.plone>
- <http://github.com/ymazzer/stanbol-buildout>

I will soon fork them on collective.
