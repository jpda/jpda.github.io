---
title: outlook 2010 — social data service and the social networking pane
description: ''
date: '2009-11-14T02:30:58.000Z'
categories: []
keywords: []
slug: >-
  /outlook-2010-social-data-service-and-the-social-networking-pane-ccc59371c461
---

I’ve been using the tech preview of Office 14 (2010) since early this year. One of the things I noticed is Outlook attempting to access the UserProfileService, (\_vti\_bin/UserProfileService.asmx) with a user-agent of Microsoft Outlook Social Connector (14.0.4514) MsoStatic (14.0.4514). It would throw a ‘not found’ exception, thus triggering an email — so I saw it a lot.

Now that I’ve gotten my hands dirty with the 2010 beta 2 bits, I’ve found what that service is here for — an entire panel dedicated to contact information — message aggregation, RSS feeds, meetings, attachments, and (my favorite) Status Updates. One of the intriguing things about Status Updates is the interestingly-named ‘Social Networking Provider’ — the only one out of the box is Sharepoint, but that’s a start.

This will probably end up being a social data series — I’m a big proponent of social networking in the enterprise, (luckily my boss is too) so this kind of stuff is HOT HOT HOT.

I’ll keep this one a dive into the social data pane — I’ll get into what I’ve found on the backend later.

This screen appears when you add a network to Outlook.

![image](https://cdn-images-1.medium.com/max/800/0*S3IQfcXEJBNJtdjt.png)

My initial thought when I see this screen is that the ‘Add Network’ button has a wealth of possibilities. Facebook, LinkedIn, etc. It’s like what xobni attempts to do, but baked in. I’ve been trying to figure out just what exactly defines these providers — an outlook add-in? A configuration file somewhere? Or will it be a whole new assembly, loaded in some kind of sandbox? With the advent of IE8 sandboxing, UAC pseudo-sandboxing and the new sandboxed solutions in SharePoint, it wouldn’t be terribly surprising to see some sort of ‘sandboxed’ solution making it’s way into other Office products.

Anyway, I’ll post later if I figure out where it gets that data.

The next thing is the actual social networking pane itself. This appears in the preview pane of Outlook, as well as messages you read or compose. All of the people you address show up in the top right corner, so you can quickly jump back and forth.

![image](https://cdn-images-1.medium.com/max/800/0*Kr9HMuZ2_AY-3daZ.png)

For each person, you get a couple tabs — Items, messages, mail, etc. Status Updates is what I’m most interested in now.

![image](https://cdn-images-1.medium.com/max/800/0*6Al_O1zDL2CsgKR0.png)

One of the first things I saw that I found quite interesting was that a simple right-click showed me we’re in nothing more than an IE\\Trident window. This is important for two reasons: easy customization & possible support for standard HTML — not word-crippled HTML like the rest of Outlook.

A simple test proves my point — that’s an embedded youtube player, within my Outlook pane. So that’s our first stop. This is great — solely because using standards, we can server whatever content we want — Flash, Silverlight, etc. (that video is awesome, by the way — search youtube for ‘giving him the business’)

![image](https://cdn-images-1.medium.com/max/800/0*y3ezpluBe-4Hdiam.png)

From what I can tell, each pane is a rendered HTML document (generated and dumped into the %temp%), with a strict doctype. A quick right-click, view properties, shows me that my assumptions are correct. You’ll find them in your local temp listed as OSCPP ##.html. In addition, it looks like it refreshes frequently — not 15 seconds or so after I took this screenshot, the html was regenerated, notepad++ started whining and my status updates pane went back to ‘No items to show in this view.’

Why is this so important — one single reason — if these tabs are extensible (I’m sure they are) — that means everything from social\\professional networking to LOB apps right within the familiar confines of Outlook. With the flexibility of full HTML (and javascript, by extension) — that means live updating, rich content and greater collaboration, both from the built-in panes as well as any extended LOB apps. Since business people love Outlook, it only makes sense to keep all the data in one spot.

More later!