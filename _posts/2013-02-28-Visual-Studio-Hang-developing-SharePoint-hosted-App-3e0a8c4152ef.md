---
title: Visual Studio Hang developing SharePoint-hosted App
description: ''
date: '2013-02-28T03:48:00.000Z'
categories: []
keywords: []
slug: /visual-studio-hang-developing-sharepoint-hosted-app-3e0a8c4152ef
---

Over here at WTF HQ, we’re trying out some App Parts. Great, except that my normally patient & docile colleague is _literally moments_ away from plunging his fist knuckle deep into his monitor.

#### App Part Architecture

App parts are pretty simple, really — an iframe web part with some initialization goo on a SharePoint page. It calls your server (provider hosted), an Azure server (autohosted) or an app web on your SharePoint infrastructure (SharePoint-hosted) and that’s about it. There’s some limited interactivity between app part & host page (and I mean _limited),_ but not much. It’s good for…well, something. I’m not quite sure yet. But I’ll let you know.

#### Srsly, why so limited?

In theory, the app part really _shouldn’t_ be so limited, but Microsoft has declared it so. Here’s how it works:

HTML5 has a nifty thing called _postMessage_ — it lets an iframe send a message to its host page. Cool, except that the event listener has to be registered in the host page. Microsoft has gone to the liberty of doing this, giving us…one event listener: resize. Yes, ladies and gentlemen, we are on the brink of technical excellence here. It wouldn’t be so bad if they’d let us register new ones, but alas, we cannot. So it really doesn’t do much.

#### That’s not the point.

So, back to Visual Studio hanging _every thirty seconds_ while developing a SharePoint-hosted app. We tried this with a few different scenarios, but any SharePoint-hosted app would just make the machine _useless._ Time for [Fiddler.](http://fiddler2.com/)

I couldn’t replicate his troubles on my machine, but rather than lose a limb in a tragic monitor-punching accident, I elected I should look in. Here’s what we found.

#### First, Intellisense Is Cray

I turned on fiddler & couldn’t get anything bad to happen…but then I started typing a `<script>` tag and this happened:

Cool, huh? Not really. Especially since a good portion of those files _don’t even exist._ But that’s not what was causing the full-blown UI hangs…

#### HTTP & HTTPS Script Tags

Writing in Office 365 is interesting, because you’re always bouncing back and forth between HTTP & HTTPS — HTTP on the public site, HTTPS on the private site _and_ when authenticated to the public site. Because of this, we use protocol-free script tags from CDNs. Like this one:

`<script src="//ajax.aspnetcdn.com/ajax/4.0/1/MicrosoftAjax.js"type="text/javascript"></script>`

Note there’s no ‘http’ or ‘https’ listed — this way, the browser will use whatever protocol you’re currently connected on. This really _is_ cool and incredibly useful.

#### WebDAV

That is, of course, until you factor in WebDAV, that old creaky protocol for fetching files over the web. Some deeper inspection of our previous `<script>` tag Fiddler explosion above gave us some insight into what was going on — not only was Visual Studio trying HTTP, it was trying WebDAV, sending OPTIONS requests to the ASP.net CDN:

The fact that we’ve all been happily coding along in MVC for months/years without this ever happening leads me to believe it has something to do with the SharePoint project type, perhaps the fact that so many files _are_ available over WebDAV with SharePoint.

#### Solution? There really isn’t one.

* Develop to a local SharePoint server, at least then the latency won’t be too annoying.
* Swap out http(s) with some server-side variables
* just use one or the other for debugging…but remember to change before publish!
