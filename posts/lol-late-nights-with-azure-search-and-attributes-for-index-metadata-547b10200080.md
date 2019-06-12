---
title: LOL — Late nights with Azure Search and Attributes for Index Metadata
description: ''
date: '2015-04-08T05:45:48.000Z'
categories: []
keywords: []
slug: >-
  /@jpda/lol-late-nights-with-azure-search-and-attributes-for-index-metadata-547b10200080
---

I’m working with a client right now on modernizing and simplifying their search to use the new Azure Search service. Sure, the examples online are fine, but I wanted to decorate my data classes with attributes dictating the indexing settings for each field. Needless to say, I ended up with something just shy of mad:

#### ???

This actually works. It’s efficiency is up for debate, but it does work. I suspect I would have had just as much luck manipulating the JSON myself, but where’s the fun in that? Here’s what I started with — a simple Azure table entity poco. Nothing too exciting, just a few fields of relevance:

This is fine — my attributes are there so I can configure indexing on the object itself. This is all well and good, but I need to actually create an index to put documents into:

This also worked well — my index schema gets created from the properties that are decorated with my attribute. The problem starts when I need to actually add the documents to the search indexer. Since I’m inheriting from TableEntity (in this case), additional properties are included (like PartitionKey, RowKey, etc). I need to only get the properties which have indexing metadata, since those match the schema of the index. Apparently, including additional properties in the document you’re submitting to the index causes the call to blow up — not just ignore the extra properties. I have to copy the relevant, decorated properties to a new object on the fly…and voila, you end up with the silliness that is above.