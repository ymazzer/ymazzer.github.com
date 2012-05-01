---
layout: post
title: "GSoC Blog week 5"
description: ""
category: 
tags: [gsoc]
---
{% include JB/setup %}

This week, I have been working on the buildout system of Plone. First I read
documentation on the system itself and how to write an add-on for Plone. Now
familiar with the buildout system, I updated stanbol-buildout and stanbol.plone
to make it working with the recent changes (move from `fise.*` to `stanbol.*`, etc
...). 

Furthermore, I corrected the places where the buildout fetch stanbol jar which
was a Dropbox where obsolete builds where standing in. It fetches now on the
jenkins of Stanbol.

