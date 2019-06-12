---
title: Applying SharePoint Themes to Apps
description: ''
date: '2013-03-01T19:44:00.000Z'
categories: []
keywords: []
slug: /applying-sharepoint-themes-to-apps-5fe1c94b31f1
---

Using the [SharePoint Chrome Control​](http://msdn.microsoft.com/en-us/library/fp179916.aspx) to get style in your app for SharePoint is all well and good, but it only does so much. You’ll get CSS imported, but there are, unfortunately, a few things that base CSS leaves out. Plus, you’ll also need to make sure & tag things appropriately so they pick up the styles. Here are a few examples:

#### ms-AccentText for Headers

This class is a great one to use because it’ll style your `h*` tags as SharePoint does normally — colors, sizes, etc. You could manually add that to each `h*`, but _ain’t no body got time for that._

#### Add them the easy way — with jQuery

In your master page of your web app, add this function (make sure it is _after_ the chrome control script has run — it usually works either way, but just to be sure…):

`$("h1, h2, h3, h4, h5, h6").addClass("ms-accentText");`

So now we’ve got some pretty headers that match SharePoint — all well and good, since (at least my goal) we want the app experience to match SharePoint as closely as possible. Why make your users learn two interfaces? It’ll only introduce confusion, which will lead to slower adoption (total speculation, but it sounds good). Next, let’s do some more styling:

#### ms-BackgroundImage

This one has a _lot_ of value for making your app appear as though it ‘belongs’ in the SharePoint site. Just slap this class on your `<body>` tag & the background image of the host web will appear.

`<body class="ms-backgroundImage" style="overflow: auto;">`

#### ms-pub-contentLayout

This is, unfortunately, one of the more difficult ones to achieve. For whatever reason, Microsoft left this one out of defaultcss.ashx (the file containing the CSS of the host web — the chrome control pulls this in or you can manually add it yourself). I’m guessing it’s because it’s a publishing-site specific class & since apps can be installed anywhere, including a class which doesn’t exist would cause drama. Oh well, we can work with it.

First, you need to make sure you’ve got a container on your page (preferably very close to where your main content is going) with the class attached:

`<div class="ms-pub-contentLayout ms-verticalAlignTop" id="contentBox" aria-live="polite" aria-relevant="all">`

Now we’ve got some hackery to get around. Since the class isn’t included in defaultcss.ashx, we’ve got to replicate it as best as we can.

```css
.ms-pub-contentLayout {  
      padding: 20px;  
      margin-top: 0;  
      padding-top: 0;  
      display: table-cell;  
      min-width: 630px;  
      opacity: 0.8;  
   }
```

This at least gives us a decent layout, however the color is missing (which is arguably the most important part of this class). There are a few ways to get close (though none are 100% reliable), but there’s a bit of a shortcut. On dark-themed pages, the color is usually white @ 80% opacity. Fortunately, on a light-theme, it wouldn’t even be noticable, so if you’re keen to just ‘deal with it,’ then this is the easiest way out.
