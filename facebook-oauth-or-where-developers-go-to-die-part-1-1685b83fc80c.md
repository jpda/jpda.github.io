---
title: "Facebook OAuth, or, where developers go to die. Part 1"
date: 2013-01-08T04:24:00.000Z
author: "John Patrick Dandison"

---

Well well, Facebook. If you read my [last post](/Blog/Post/4/Connecting-to-OAuth-Services-from-a-SharePoint-App), I’m attempting to get a SharePoint app connected to different services for blog syndication. Twitter &amp; LinkedIn were walks in the park. Facebook was a walk in the park as well, however I suspect it is the kind of park with large spikes &amp; vicious animals.

Anyway, let’s begin.

I elected to use the Facebook C# SDK for Facebook. The examples made authentication look *so* easy that I was sold. In a classic ‘AdventureWorks’ scenario, where the sample really never dictates normal usage, I found myself face deep in facebook OAuth errors, almost all of them specific to the ‘redirect_uri’ parameter:

_Error validating verification code. Please make sure your redirect_uri is identical to the one you used in the OAuth dialog request._

#### Problem?

After getting trolled by facebook, I decided to try some things to fix this:

*   Immediate window: redirect_uris all matched.
*   Fiddler + Notepad: redirect_uris all matched.
*   10 e14 different variations of Url.Encode

Nada.

#### Lots of yapping, not a lot of fixing

After realizing that I wouldn’t be so fortunate as to have this simply work, I started down a new path. Thanks to one of the bazillions of stackoverflow posts, I found the ‘state’ parameter that facebook allows during oauth transactions. Surely this is the ticket?

After all, the authorization code uses that state parameter to transfer around a cross-site forgery token, so it must be working…I added two new properties to the anonymous object the fb SDK was already using for state, and sure enough they came back with my request.

So that’s the first part — at least now facebook is sending me back the proper tokens. But how to make the application aware? That means venturing into the bowels of TokenHelper.cs.
