---
title: 'Let’s talk about auth, ba-by (or, Headless Auth to Office 365)'
description: ''
date: '2013-12-13T19:29:14.000Z'
categories: []
keywords: []
slug: /@jpda/lets-talk-about-auth-ba-by-or-headless-auth-to-office-365-480ccbd45c24
---

Authentication in SharePoint Online…now that’s a topic that’s been beaten all over the internet. The premier source for doing-what-you-need not doing-what-you’re-told​ is probably Wictor Wilen’s work on SharePoint Online, active authentication and [yanking cookies off of requests.](http://www.wictorwilen.se/Post/How-to-do-active-authentication-to-Office-365-and-SharePoint-Online.aspx) This is perfect for client/mobile/non-browser apps that need to do things with SharePoint Online.

Here at WTFHQ we use a lot of services. \*Lots\* of services. In fact, I can’t think of much that doesn’t call back into some service somewhere. How else would you ever do things? For example, since there are no timer jobs in SharePoint Online, you might have a scheduled task that runs on an internal server somewhere or a SQL job that needs to do some \*stuff\* to a SharePoint Online tenancy.

#### Active Auth vs. Service Accounts

Wictor’s way is cool and all, especially if you’re doing end-user type apps where someone will open the app, need to authenticate, a browser is shown, user logs into Office 365, cookies are yanked, app can now make calls on behalf of the user until that session/those cookies expire. In fact, if you’re doing an end-user app that requires user authentication, **stop reading this now.** You need to do exactly what Wictor is prescribing. In fact, if you \*don’t\* do it his way, your app will be bad and you should feel bad.

Doesn’t really work too well for headless calls, though. Imagine if the Headless Horseman had to announce his arrival? It wouldn’t go too well.

So what’s a services developer to do? Fortunately, someone else has already figured it out.

#### Microsoft.SharePoint.Client.SharePointOnlineCredential

You’ll see this class thrown around a lot in PowerShell circles. Not that it’s a bad thing, it just _is._

It is exactly what it looks like — a username/password pair of SharePointOnlineCredential (who knew?). Using this, you can get an authenticated ClientContext — from there it’s the same ClientContext code you love to _PropertyOrFieldNotInitializedException_

#### Usage

Here’s a quick and dirty sample I threw together yesterday for some peeps at work. It’s so easy, a VB developer could do it:

var sharePointUrl = inputArgs\["Url"\];  
var password = inputArgs\["Password"\];  
var username = inputArgs\["Username"\];

Console.Write("Connecting to {0} as {1}...", sharePointUrl, username);

var securePassword = new SecureString();  
password.ToList().ForEach(securePassword.AppendChar); //don't hate on my greedy memory usage: string --> list --> securestring

var ctx = new ClientContext(sharePointUrl) { Credentials = new SharePointOnlineCredentials(username, securePassword) };  
var web = ctx.Web;

ctx.Load(web);  
ctx.ExecuteQuery();

Console.WriteLine("done.");

Console.WriteLine("The web at {0} is named \\"{1}\\" and here are some of the lists: ", sharePointUrl, web.Title);  
ctx.Load(ctx.Web.Lists, x => x.Include(y => y.Title, y => y.ItemCount, y => y.LastItemModifiedDate));  
ctx.ExecuteQuery();  
foreach (var l in ctx.Web.Lists.ToList())  
{  
  Console.WriteLine("List {0} has {1} items and was last modified at {2}.", l.Title, l.ItemCount, l.LastItemModifiedDate);  
}