---
title: Cross site collections with SharePointContextProvider
description: ''
date: '2013-11-19T22:17:47.000Z'
categories: []
keywords: []
slug: /cross-site-collections-with-sharepointcontextprovider-e14cb16e04e
---

SharePoint apps are pretty site-scoped — from the installation mechanism to the programming guidelines, it’s all pretty geared towards Joe in Accounting installing an app that’ll let him process expense reports faster or Sue in sales being able to track client contact faster. Which is fine. And a bit boring.

#### SharePointContextFilterProvider

You could roll your own abstractions of SharePoint Apps’ OAuth flow, if you’re into that sort of thing (not that there’s anything wrong with that). But if you have things to do, you might as well use the SharePointContextProvider that’s rolled into the tools for VS 2013 (aka SharePointContextFilter ages ago in the VS 2012 version of the tools). These abstract the OAuth flow in a way that you really don’t have to do much, just ask for a context and let the provider take care of the rest.

#### Crossing the Site Collection Boundary

So yeah yeah, blah blah blah, enough yapping, give me what I’m here for — ok, so it’s pretty simple. All we really want to do is create a new context and override the SPHostWebUrl property. Pretty simple, huh?

In the SharePointContext abstract class, add this method:

```c#
public SharePointContextCreateSharePointContext(HttpRequestBase httpRequest, string targetWebUrl)  
{  
 var ctx = CreateSharePointContext(httpRequest);  
 return CreateSharePointContext(new Uri(targetWebUrl), ctx.SPAppWebUrl, ctx.SPLanguage, ctx.SPClientTag, ctx.SPProductNumber, httpRequest);  
}
```

Now you’re pretty much done ~~breaking~~ modifying `SharePointContext`.

#### Usage is easy too.

Instead of using SharePointContextProvider.Current.GetCurrentSharePointContext, you’re going to hit it with a SPCP.Current.CreateSharePointContext, sending in your HttpContext & the target host web URL:

```c#
public ActionResult DoSomethingInAnotherSiteCollection()  
{  
  using (var superCtx = SharePointContextProvider.Current.CreateSharePointContext(HttpContext.Request, "https://jpda.sharepoint.com/sites/apps").CreateUserClientContextForSPHost())  
  {  
    var web = superCtx.Site.RootWeb;  
    superCtx.Load(web, x => x.Title, x => x.Url);  
    superCtx.ExecuteQuery();  
    return Json(new { Result = "OK", Message = string.Format("Site collection {0} is at {1}...where are you?!", web.Title, web.Url) }, JsonRequestBehavior.AllowGet);  
  }  
}
```

#### Before you flex your CTRL-C/CTRL-V muscle, there’s more.

There is _one_ more thing — since we’re creating an entirely new context, you’ll need all of the information that was on the original request — like Language, Host & App web urls and most importantly, the context token. Alternatively, you could override the CreateSharePointContext method and pass all that stuff in as strings — whatever floats your boat. I don’t like that, because lots of parameters make my head hurt.

Here’s a hacked (shitty) bit of javascript that calls my method with all of the headers. You could do this server side (I mean, to be pedantic, this *is* server side since it’s razor tokens but I digress — you get the point) — ultimately, your HttpContext.Request object needs to have all of these tokens in it or you need to add a method that accepts them all as strings.

```js
var sph = "@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPHostUrl";  
var spa = "@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPAppWebUrl";  
var spl = "@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPLanguage";  
var spn = "@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPProductNumber";  
var spc = "@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPClientTag";  
var spct = "@(((SharePointAcsContext) SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current)).ContextToken)";

$(function() {  
  $.ajax({  
    url: "/Home/DoSomethingInAnotherSiteCollection",  
    data: { SPHostUrl: sph, SPAppWebUrl: spa, SPLanguage: spl, SPClientTag: spc, SPProductNumber: spn, AppContextToken: spct },  
    dataType: "json",  
    success: function(data) {  
      $("#cross-site").text(data.Message);  
    },  
    error: function(xhr, status, errorCode) {  
      $("#cross-site").text(errorCode);  
    }  
});  
});
```

And here's what you'll see.

This app is installed in one site collection, but I'm getting the title of another site collection via CSOM.
