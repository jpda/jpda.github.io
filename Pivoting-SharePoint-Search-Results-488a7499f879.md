---
title: "Pivoting SharePoint Search Results"
date: 2011-02-15T03:35:00.000Z
author: "John Patrick Dandison"

---

I’ve spent some time recently working on a side, pet project for SharePoint search results. I haven’t had nearly the time to dedicate to it that I’d like, but I figured I’d share anyway.

First, check out Live Labs Pivot (now called Silverlight Pivotviewer) at [http://www.microsoft.com/silverlight/pivotviewer/](http://www.microsoft.com/silverlight/pivotviewer/)– it’s a great tool for interacting with large amounts of data.

One of the complaints we hear frequently in our enterprise relate to search result relevancy and manageability. A wealth of relevant results is pretty worthless if it’s a bear to deal with, so Pivot seemed like a natural choice.

Here we have search results returned as a Pivot collection, but the CXML (Collection XML) and DZI (DeepZoom tiles) are built and streamed out on the fly, rather than persisted to disk somewhere.

I’m not going to get into the specifics of the implementation, as it’s just PoC and I haven’t updated it yet for 2010. I’m hoping to get some time to dedicate to this and a FAST Search server for more dynamic thumbnails.

More to come.




![image](http://jpd.ms/wp-content/uploads/migrated/image_thumb_18.png)
