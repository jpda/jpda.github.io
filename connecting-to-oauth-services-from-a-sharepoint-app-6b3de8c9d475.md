---
title: "Connecting to OAuth Services from a SharePoint App"
date: 2013-01-08T04:05:29.000Z
author: "John Patrick Dandison"

---

One of the first things I’m working on in this new SharePoint App world is an auto-syndicator to publish your SharePoint blog posts to Facebook, LinkedIn &amp; Twitter. Sounds simple, in theory…

#### Creating your SharePoint App — there’s nothing new here.

If you’re not familiar with developing SharePoint apps yet, it’s probably best to head on over to MSDN and get deep on those dox. There are a _lot,_ of varying quality. In short, you’ll have a frustrating couple of days…but you’re a SharePoint developer, so this should be nothing new for you.

SharePoint apps use OAuth for getting permissions from SharePoint &amp; figuring out who you are. When an app is ‘installed,’ the installing user grants your app those permissions &amp; that’s about it, from the user side. _For you,_ however, the fun has only just begun. Whenever a user comes to your app, they’ll come with a variety of query strings &amp; tokens. These include things like **SPHostUrl** &amp; **SPAppToken**. I call these two out explicitly because they’re probably the most important (there are others, but irrelevant for now). **SPHostUrl** contains the URL of the user’s SharePoint site where he/she has ‘installed’ your app (i.e., granted permissions). The AppToken is the OAuth token that lets you get back to authenticated SharePoint resources. Here’s the thing — if you need _anything_ SharePoint related (current user, list data, etc), you’re going to need this. On every request. Where you store it is up to you, but remember that things like Session &amp; ViewState are lame and you shouldn’t use them. _Ever._You can find some of what I think [here](/Blog/Post/3/Apps-for-SharePoint,-MVC--amp;-OAuth--Identity-Hell--).

#### SharePointContextFilter for MVC

Enter the [SharePointContextFilter](http://social.msdn.microsoft.com/Forums/en-US/appsforsharepoint/thread/fa15960f-340d-4e69-a703-47b607278da9), by Maxim Lukiyanov. It will make your life _dramatically_ better when writing MVC apps in SharePoint. It does what TokenHelper does not — any method/controller you decorate with this filter will make sure there is a token, find it, get it, create a SharePoint Context &amp; lots of other fun stuff. If there isn’t one, it even redirects you back to SharePoint to get a fresh one. How nice.

#### Facebook. LinkedIn. Twitter. OAuth for days, son.

Now to the point. There are lots of ways to connect to different services &amp; I’m actually using three different ways. LinkedIn &amp; Twitter are pretty straightforward. Facebook is a PITA, so I’ll delve into that on another post.

#### Connecting to Services

[DotNetOpenAuth](http://www.dotnetopenauth.net/) is the _de facto_ standard for OAuth in .net. It’s used in a lot of projects, so stop trying to be a unique butterfly &amp; use it. You don’t want to reimplement OAuth, no matter how arrogant of a developer you may be.

But I’ll back up for a second. When you want to authenticate to a service, you’ve gotta A) sign up for a dev key (usually you’ll get a key &amp; secret) and B) have some URL in your app that can receive and process the return message. So the flow’s like this:

Dude goes to your app à Clicks ‘Login with ‘ à Redirects to an authentication endpoint at that service with some parameters (like your key &amp; secret) à gets redirected back to your application with some more, different parameters. Pretty simple flow, really. In our SharePoint scenario, however, this changes slightly. If you want to know anything about the user (like, say, to bind their service token to their SharePoint username/identifier), you need to query SharePoint for that information…and in order to query SharePoint, you need an SPAppToken…and to get an SPAppToken, you need to hit SharePoint first, then get redirected into your app (or save the SPAppToken in some sort of temporary storage when the user first hits your app).

#### Twitter

I’ll start with Twitter. I decided to use [linq2twitter](http://linqtotwitter.codeplex.com/), since it’s fairly robust &amp; does what I want it to (plus it’s better, because LINQ). It has its own authentication wrappers for Twitter that seem to handle long redirect URIs pretty well.

Here’s the snippet for linq2twitter. It’s pretty simple to implement.

In the markup:

&lt; a  
 href=”@Url.Action(“ConnectToTwitter”, new { SPHostUrl = SharePointContext.Current.SPHostWebUrl, SPAppToken = SharePointContext.Current.ContextTokenString })”  
 …/&gt;

And the controller:

public  
 ActionResult ConnectToTwitter()

{

var credentials = new  
 InMemoryCredentials();

if (credentials.ConsumerKey == null || credentials.ConsumerSecret == null)

{

credentials.ConsumerKey = ConfigurationManager.AppSettings[“TwitterConsumerKey”];

credentials.ConsumerSecret = ConfigurationManager.AppSettings[“TwitterConsumerSecret”];

}

var auth = new  
 MvcAuthorizer { Credentials = credentials };

auth.CompleteAuthorization(Request.Url);

if (auth.IsAuthorized)

{

//You’ll have a SharePoint token, so get an SPContext from the SPContextFilter &amp; query for the user

}

if (!auth.IsAuthorized)

{

var specialUri = Request.Url;

return auth.BeginAuthorization(specialUri);

}

return RedirectToAction(“Index”, “Home”);

}

Why don’t you have to play with the tokens? Besides the fact it’ll make you go blind (or so I hear), the SharePointContextFilter is automatically adding the querystring parameters necessary — **SPAppToken** &amp; **SPHostUrl.** We can verify in Fiddler. See all that mess down there? It’s the callback URL with all SharePoint-specific querystring properties encoded appropriately. On the return trip, my app has a SharePoint token, so it can figure out what SharePoint user you are, but it also has your Twitter token as well, for making subsequent requests to Twitter. Pretty nifty.

GET /oauth/request_token HTTP/1.1

OAuth oauth_callback=”

https%3A%2F%2Flocalhost%3A44308%2FIdentity%2FConnectToTwitter%3F

**SPHostUrl**%3D%26

**SPAppToken**%”,

oauth_consumer_key=””,

…other_oauth_headers…

#### LinkedIn

LinkedIn is pretty similar. Not too much drama, although DNOA exposes more of the inner workings of OAuth — for instance, I have to store the request token, then replace it with the access token. You’ll have to build your own URL, too, appending the SPHostUrl &amp; SPAppToken parameters yourself.

Kinda like this:

public  
 ActionResult ConnectToLinkedIn()

{

var redirectParams = string.Format(“?SPAppToken={0}&amp;SPHostUrl={1}”, SharePointContext.Current.ContextTokenString, SharePointContext.Current.SPHostWebUrl);

var serviceProvider = Utility.GetLinkedInServiceDescription();

var consumer = new  
 WebConsumer(serviceProvider, _tokenManager);

var authUrl = new  
 Uri(string.Format(“{0}://{1}{2}/{3}”, Request.Url.Scheme, Request.Url.Authority, ConfigurationManager.AppSettings[“LinkedInCallback”], redirectParams));

consumer.Channel.Send(consumer.PrepareRequestUserAuthorization(authUrl, new  
 Dictionary&lt; string, string&gt;() { { “scope”, “rw_nus” } }, null));

return  
 null;

}

But that’s really about it. You can use (at least for LinkedIn &amp; Twitter) pretty standard methods for connecting users to your app. Facebook is…facebook, and requires another post for the drama it took to get working. Coming up.
