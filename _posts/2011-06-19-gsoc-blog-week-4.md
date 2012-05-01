---
layout: post
title: "GSoC Blog week 4"
description: ""
category: 
tags: [gsoc]
---
{% include JB/setup %}

This week has been a big refactoring work on fise.client, fise.plone and
fise-buildout, to match the stanbol renaming. The mother class is now
StanbolCommunicator. Anyway the API used to access Engine and Store class has
not changed much :

Initialize::

    >>> from stanbol.client import Stanbol
    >>> stanbol = Stanbol('http://localhost:8080/')

Use the engines::    

    >>> somedoc = u"This is an example text."
    >>> stanbol.engines(somedoc)
    <xml...>

    >>> stanbol.engines(somedoc,
    >>> format='rdfjson')
    jsonresponse

Use the store, first store content (only plain text is accepted for now)::

    >>> id = 'test123'
    >>> stanbol.store.content[id]
    >>> = payload

Next get the text back::    

    >>> stanbol.store.content[id]
    u"This is an example text."

Then get the metadata::

    >>> stanbol.store.metadata(id)
    <RDF>

And Stanbol special feature: Get an HTML page about the content::

    >>> stanbol.store.page(id)
    <HTML>
