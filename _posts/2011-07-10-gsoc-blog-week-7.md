---
layout: post
title: "GSoC Blog week 7"
description: ""
category: 
tags: [gsoc]
---
{% include JB/setup %}

This week, I wanted to develop the `system/console` python API, but this is not
really relevant since the console is more to manage the OSGI server than stanbol
itself.

That's why I decided to delay this part, and maybe to forget about it, and to
concentrate on the Plone integration. I developped the control panel part
available on github and began to develop a JavaScript library to connect to
Stanbol.

The point of the latter is to be able to fetch metadata directly from the client
and fill the `category/tag` field automatically. This will be the PoC of Stanbol
Plone integration. Since it's not working yet I didn't push it on github. But I
will certainly push it during next week and publish a demonstration instance of
Plone featuring early Stanbol integration.
