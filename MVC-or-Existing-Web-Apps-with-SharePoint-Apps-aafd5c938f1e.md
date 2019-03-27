---
title: "MVC or Existing Web Apps with SharePoint Apps"
date: 2013-03-08T07:28:00.000Z
author: "John Patrick Dandison"

---

I’ve seen a bit of search traffic heading this way about adding an MVC project or an existing web app to a SharePoint app — good news: it’s easy.

This is so simple you’ll want to slap someone. Say you’ve already got a bitchin’ web app, but now you want to make it a SharePoint app…OR, let’s say you want to make ​your first SharePoint app, but you don’t want to use *webforms* (because, let’s face it, you’re not 187 years old). Let’s start with a new app.

#### Step One: Exorcising Web Forms

When you create a new SharePoint app, you’re stuck with webforms by default. Webforms, that old man in the wheelchair who can say nothing, yet bury you in contempt. That’s cool though, because it’s easy to wheel that chair straight out the window.

*   First, add a new project to your solution — MVC, whatever.
*   Next, click the SharePoint App project (not the webforms project, the SharePoint app project itself — it’ll have AppManifest.xml) — and hit F4 (open the properties pane)
*   Peep the last property on the page: Web Project — this little drop down will show you all the web projects in your solution, regardless of webforms or MVC. Check it out:

#### Step Two: Binding your new (or existing) web project to the SharePoint App

The _real_ magic happens when you select the new web project. You’ll get a prompt (or two), but the important one is this one:

Remember our good friend TokenHelper.cs? This is how he gets in there, not to mention a few other key ingredients for our SharePoint project to F5-debug &amp; publish. For instance, ClientId &amp; ClientSecret get added to the web.config, plus the ~remoteAppUrl token in your AppManifest.xml will get populated properly (for provider-hosted apps).

#### B-b-b-but you said I could use an existing app!

Indeed you can — it’s essentially the same process. Open your existing web app’s solution (or create a new one, whatever) &amp; add a SharePoint App project. Once it’s in there, open the properties pane &amp; get to switchin’.

### _One caveat!_

Since SharePoint-hosted apps are just that — hosted in SharePoint, they’ve gotta be webforms. Sucks, I know, but _c’est la vie._

#### Tomorrow I’ve got some stuff on deck for using Azure Cloud Services (as opposed to Azure Websites) as a SharePoint app. It’s a bit of a PITA, but it _can_ be done.
