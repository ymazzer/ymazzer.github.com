---
layout: post
title: "GSoC Blog week 8 and 9"
description: ""
category: 
tags: [gsoc]
---
{% include JB/setup %}

I did not post anything for two weeks, which is far too long but results are
coming. The tag completion and RDF text enhancements are almost done. I am
facing some configurations bugs which implies it does not work yet. From now on,
the plone integration is able to query a Stanbol instance using the
`stanbol.client` python library and the javascript part is able to communicate
with the python module using `jquery.pyproxy`. I didn't manage to make the control
panel work for now, I certainly missed something in the configuration part since
Plone is not able to find my portal_stanbol object. Once it is done I will
publish a working demo.
