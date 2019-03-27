---
title: "Cross site collections with SharePointContextProvider"
date: 2013-11-19T22:17:47.000Z
author: "John Patrick Dandison"

---

SharePoint apps are pretty site-scoped — from the installation mechanism to the programming guidelines, it’s all pretty geared towards Joe in Accounting installing an app that’ll let him process expense reports faster or Sue in sales being able to track client contact faster. Which is fine. And a bit boring.

#### SharePointContextFilterProvider

You could roll your own abstractions of SharePoint Apps’ OAuth flow, if you’re into that sort of thing (not that there’s anything wrong with that). But if you have things to do, you might as well use the SharePointContextProvider that’s rolled into the tools for VS 2013 (aka SharePointContextFilter ages ago in the VS 2012 version of the tools). These abstract the OAuth flow in a way that you really don’t have to do much, just ask for a context and let the provider take care of the rest.

#### Crossing the Site Collection Boundary

So yeah yeah, blah blah blah, enough yapping, give me what I’m here for — ok, so it’s pretty simple. All we really want to do is create a new context and override the SPHostWebUrl property. Pretty simple, huh?

In the SharePointContext abstract class, add this method:
`public SharePointContextCreateSharePointContext(HttpRequestBase httpRequest, string targetWebUrl)  
{  
 var ctx = CreateSharePointContext(httpRequest);  
 return CreateSharePointContext(new Uri(targetWebUrl), ctx.SPAppWebUrl, ctx.SPLanguage, ctx.SPClientTag, ctx.SPProductNumber, httpRequest);  
}`

Now you’re pretty much done  
breaking modifying SharePointContext.

#### Usage is easy too.

Instead of using SharePointContextProvider.Current.GetCurrentSharePointContext, you’re going to hit it with a SPCP.Current.CreateSharePointContext, sending in your HttpContext &amp; the target host web URL:
`public ActionResult DoSomethingInAnotherSiteCollection()  
{  
  using (var superCtx = SharePointContextProvider.Current.CreateSharePointContext(HttpContext.Request, &#34;https://jpda.sharepoint.com/sites/apps&#34;).CreateUserClientContextForSPHost())  
  {  
    var web = superCtx.Site.RootWeb;  
    superCtx.Load(web, x =&gt; x.Title, x =&gt; x.Url);  
    superCtx.ExecuteQuery();  
    return Json(new { Result = &#34;OK&#34;, Message = string.Format(&#34;Site collection {0} is at {1}...where are you?!&#34;, web.Title, web.Url) }, JsonRequestBehavior.AllowGet);  
  }  
}`

#### Before you flex your CTRL-C/CTRL-V muscle, there’s more.

There is  
_one_ more thing — since we’re creating an entirely new context, you’ll need all of the information that was on the original request — like Language, Host &amp; App web urls and most importantly, the context token. Alternatively, you could override the CreateSharePointContext method and pass all that stuff in as strings — whatever floats your boat. I don’t like that, because lots of parameters make my head hurt.

Here’s a hacked (shitty) bit of javascript that calls my method with all of the headers. You could do this server side (I mean, to be pedantic, this *is* server side since it’s razor tokens but I digress — you get the point) — ultimately, your HttpContext.Request object needs to have all of these tokens in it or you need to add a method that accepts them all as strings.
`var sph = &#34;@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPHostUrl&#34;;  
var spa = &#34;@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPAppWebUrl&#34;;  
var spl = &#34;@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPLanguage&#34;;  
var spn = &#34;@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPProductNumber&#34;;  
var spc = &#34;@SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current).SPClientTag&#34;;  
var spct = &#34;@(((SharePointAcsContext) SharePointContextProvider.Current.GetSharePointContext(HttpContext.Current)).ContextToken)&#34;;``$(function() {  
  $.ajax({  
    url: &#34;/Home/DoSomethingInAnotherSiteCollection&#34;,  
    data: { SPHostUrl: sph, SPAppWebUrl: spa, SPLanguage: spl, SPClientTag: spc, SPProductNumber: spn, AppContextToken: spct },  
    dataType: &#34;json&#34;,  
    success: function(data) {  
      $(&#34;#cross-site&#34;).text(data.Message);  
    },  
    error: function(xhr, status, errorCode) {  
      $(&#34;#cross-site&#34;).text(errorCode);  
    }  
});  
});``And here&#39;s what you&#39;ll see.``This app is installed in one site collection, but I&#39;m getting the title of another site collection via CSOM.`
