---
title: I migrated my blog this weekend
description: ''
date: '2014-05-26T04:55:52.000Z'
categories: []
keywords: []
slug: /i-migrated-my-blog-this-weekend-ff451ecb3655
---

Needless to say, you’ve made it. I decided to move both johndandison.com/blog & wtfsharepoint.com here _and_ to consolidate the content. There’s still some stuff lagging behind, but I think for the most part my links are properly sending 301s so _hopefully_ it won’t be too bad. I decided to use BlogEngine.net a long time ago, thinking I was familiar with .net and might extend it one day. Three years later and all I did was change the theme…

So I’m off to wordpress now. It’s been pretty nice, you’ll notice none of my old posts are categorized, but hopefully I’ll get around to it.

My 301s are all coming from an Azure app, which has taken over duties for johndandison.com. I’ll post that (relatively awful, but simple) code up to github later, I suppose, if there’s interest. It’s a simple lookup in table storage, old post → new post, do a 301, then update statistics. At $9/mo I’ll probably just leave that app up for a bit until Google & Bing have managed to reindex me.

There were two things that I didn’t expect, one has been solved and one hasn’t.

#### 400 Bad Request using a URL as a RowKey

This was surprising but at the same time not.

I definitely wanted to use the Source URL as the RowKey. Since I could do direct lookups in table storage (PartitionKey + RowKey), this would be the fastest.

A URL is full of all kinds of stuff a REST service wouldn’t want pumped into it — so I decided to base64 encode my URLs, which would produce me a nice chunk of valid text I could stuff into the RowKey. Or so I thought.

Occasionally, base64 strings include a slash (‘/’), which is definitely _not_ valid for a RowKey. Fortunately a quick answer emerged from someone else with the same problem — base64 encode, swap out the / for \_ and do the reverse when pulling it back out. Brilliant!

#### MVC Routing + Running Managed Handlers for All Requests

I have some content folders in my site, mostly with some code assemblies/files, pictures, etc. I moved these to the new folder of my new domain and wanted my MVC app to issue 301s (although I’m not sure how useful a 301 is in a content case) and redirect to the destination. Definitely temporary, since the content links need to all be updated, but still a good safety net so my years of trash strewn about the internet continues to work.

The problem became with the routing. Making a request for [http://johndandison.com/stuff/sbs/johndandison.SBS.StorageProvider.dll](http://johndandison.com/stuff/sbs/johndandison.SBS.StorageProvider.dll) never hit my MVC app, it just 404'd. A 404 would be expected if I wanted StaticFileHandler to serve it, since the file doesn’t exist any longer on johndandison.com, which is the Azure app.

I tried forcing everything to run using runAllManagedModulesForAllRequests but my app still wasn’t being hit. Not sure if it’s a route problem or an IIS problem, but it’s definitely annoying. Since I’m on Azure web sites I haven’t really dug into it much more. I’m tempted to just write a quick ‘n dirty Azure worker role to listen for requests and spit out the redirect, but it’s a holiday weekend and I just haven’t gotten around to it.

Anyway — if you find something broken, let me know!
