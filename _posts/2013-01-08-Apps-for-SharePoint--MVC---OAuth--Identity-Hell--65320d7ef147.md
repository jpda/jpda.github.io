---
title: 'Apps for SharePoint, MVC & OAuth. Identity Hell.'
description: ''
date: '2013-01-08T03:11:24.000Z'
categories: []
keywords: []
slug: /apps-for-sharepoint-mvc-oauth-identity-hell-65320d7ef147
---

Ok, so it’s not really that bad. Just being a little dramatic. But it does suck.

But it’s also about the only way to do what it is we’re trying to do — NTLM would kinda do it, but of course, isn’t usable over teh interwebs.

#### Would you like a token, sir?

No thanks, I’m full. They all use OAuth for application & user permissions. There are lots of people _way_ smarter than me who have written extensively on this topic, so you’d do yourself a favor to read about that stuff elsewhere. Basically, OAuth uses a lot of tokens & a lot of redirect pinball to ultimately tell whatever is asking who you are and what the app can do…and there are _lots_ of tokens.

#### WebForms Suck

When you create your first SharePoint app (App for SharePoint 2013 project template in VS2012), you’ll get two projects in your solution — the SharePoint ‘app’ & the actual web app that does whatever brilliant thing you’ve decided to do. Unfortunately, in a decidedly unbrilliant move, the default project is webforms. Srsly, WTF. First things first, create a modern project…like MVC4. Then find your SharePoint project, open the properties pane (F4 for your keyboard types) & find ‘Web Project.’ Pick the new MVC4 one. You’ll be asked questions. Click yes until they go away. Now you’ve got an MVC4 SharePoint 2013 app.

Curious as to what you clicked ‘Yes’ to a bunch of times? Of course you are. In a move that almost redeems the webforms-by-default move, the Yes boxes you hastily clicked through added some SharePoint goo to your MVC project, most notably, TokenHelper.cs (you _are_ using C#, correct…?). This class is included in the default webforms project (along with other relics from 2002, like a pirated copy of _8 Mile_), but the SharePoint project template will recreate it in any web project you pick through this properties pane. So on the one hand, Microsoft is putting you in a time machine, forcing you back to the dark days of web forms…but at least they’re letting you CTRL-Z that decision pretty quickly.

#### TokenHelper.cs

Here’s where the meat of the OAuth pinball takes place. When a user has installed your app (which is really just granting permissions for your app to do stuff and, optionally, hosting it within their own Azure instance), it’ll show up like everything else does in SharePoint. ‘Starting’ the app is as involved as clicking it, at which point the user is shuttled over to your app with a boatload of tokens. These tokens provide access to query back into the SharePoint environment (via the CSOM), so they are pretty important. It’s probably a good idea to store them somewhere, since you’re going to need them anytime you need a SharePointContext (which, unless your app is lame, is going to be frequently).

#### MVC & SharePointContextFilter

A really awesome chunk of code has made its way onto the internet. It’s called ‘SharePointContextFilter’ & it’s an MVC filter that requires a token be in the request on any controller or method you decorate. Find it here — [SharePointContextFilter](http://social.msdn.microsoft.com/Forums/en-US/appsforsharepoint/thread/fa15960f-340d-4e69-a703-47b607278da9), by Maxim Lukiyanov. It’s a lifesaver, so I’d suggest you download it. To use it, drop \[SharePointContextFilter\] on any controller or method that needs a SharePointContext for querying. It handles all of the interaction with the TokenHelper, so unless you are doing something beyond basic SharePoint queries, it may be sufficient enough to handle the majority of your app.

Put it all together & what do you have? A SharePoint 2013 MVC app. Now you can actually get to work making your app do whatever it is that you want it to do. Probably be pretty simple to convert an existing MVC app to use SharePointContext this way too.