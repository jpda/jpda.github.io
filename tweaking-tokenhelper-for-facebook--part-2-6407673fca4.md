---
title: "Tweaking TokenHelper for Facebook. Part 2"
date: 2013-01-08T04:50:04.000Z
author: "John Patrick Dandison"

---

We’re here for the last piece of scandalous social connectivity puzzle. Modifying TokenHelper &amp; the SharePoint context filter to get ’em right with facebook.

I’m going to post a couple of new methods here &amp; I haven’t refactored yet…so at least take a _cursory_ glance before copying/pasting into your code.

#### What to Do

Here’s the gist — Facebook returns the state parameter, so now we gotta go find that parameter, pick it out, decode it &amp; get it into the SharePointContextFilter pipeline.

Here’s the code that starts it off. Controller side:

var state = Convert.ToBase64String(Encoding.UTF8.GetBytes(_fb.SerializeJson(new { returnUrl = returnUrl, csrf = csrfToken, SPAppToken = spAppToken, SPHostUrl = spHostUrl })));

var redirectUri = new  
 Uri(Request.Url.Scheme + “://” + Request.Url.Authority + ConfigurationManager.AppSettings[“FacebookCallback”]);

var fbLoginUrl = _fb.GetLoginUrl(

new

{

client_id = ConfigurationManager.AppSettings[“FacebookKey”],

client_secret = ConfigurationManager.AppSettings[“FacebookSecret”],

redirect_uri = redirectUri.ToString(),

response_type = “code”,

scope = “user_about_me,publish_actions,read_stream,manage_pages”, state

});

The return side is where we have to get our hands dirty, namely, in the TokenHelper.

#### Dirty

I originally wanted to do this in the SharePointContextFilter, to leave the TokenHelper as pristine as possible. I still think I could do this, but I got lazy.

Since the HttpRequest.Params &amp; QueryString collections are read-only, you’ll have to copy them to a new one. So if you wanted to do this in SharePointContextFilter, you could.

First, change GetContextTokenFromRequest. Since HttpRequest is just a specialized NameValueCollection, I created a new method called GetTokenFromNameValueCollection. This could, in theory, be just an overload of GetContextTokenFromRequest.

public  
 static  
 string GetContextTokenFromRequest(HttpRequest request)

{

if (request.Params.AllKeys.Contains(“state”))

{

return GetContextTokenFromState(request);

}

return GetTokenFromNameValueCollection(request.Params);

}

Next, here’s GetTokenFromNameValueCollection.

public  
 static  
 string GetTokenFromNameValueCollection(NameValueCollection nvc)

{

string[] paramNames = { “AppContext”, “AppContextToken”, “AccessToken”, “SPAppToken” };

return (from paramName in paramNames where !string.IsNullOrEmpty(nvc[paramName]) select nvc[paramName]).FirstOrDefault();

}

Here’s the real meat of what we want to do — we want to get the ContextToken out of the State parameter, which is a base64 string. Since we can’t modify the HttpRequest, we need to copy it to a new NameValueCollection, then we get read/write &amp; can tack on our new stuff.

public  
 static  
 string GetContextTokenFromState(HttpRequest request)

{

var state = Encoding.UTF8.GetString(Convert.FromBase64String(HttpContext.Current.Request.Params[“state”]));

dynamic stuff = JsonConvert.DeserializeObject(state);

var nvc = new  
 NameValueCollection(request.Params)

{

{“SPHostUrl”, stuff.SPHostUrl.Value},

{“SPAppToken”, stuff.SPAppToken.Value}

};

return GetTokenFromNameValueCollection(request.Params);

}

As I’ll probably be refactoring this in the next few days, I may update this code. Primarily to be a bit more robust — just looking for a ‘state’ parameter is a little dangerous. It might be best to check some of the other request values (like, whether the requested URL matches what I know to be my facebook oauth receiver).
