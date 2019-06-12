---
title: Headless Azure AD User Creation
description: ''
date: '2014-09-05T01:17:52.000Z'
categories: []
keywords: []
slug: /headless-azure-ad-user-creation-6e552e1f5edb
---

If you’ve spent any time with the Azure Graph API, it’s pretty sweet. Federated identity for the masses, with almost _zero_ drama. Up until now I was mostly doing logins, queries, etc. with Azure AD, but for my latest project, I need to create both new domains _and_ new users in those domains. I haven’t tackled creating new domains yet, because that looks like it’s going to be a royal PITA (automating powershell? ick) — but I kicked down the user path today. Went pretty well, until I got stopped cold adding a user.

#### Here’s some code

Adding a user with [ADALv2](http://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/) + the [Active Directory Graph Client](http://www.nuget.org/packages/Microsoft.Azure.ActiveDirectory.GraphClient/) is pretty easy. Both are NuGet packages and simplify the process considerably. You can also post the JSON yourself, which you can find [here](http://msdn.microsoft.com/en-us/library/dn130117.aspx) on MSDN.

But I’m using ADGC, so here’s a quick snip of the required fields you’ll need to get a user created:

```c#
var gc = new GraphConnection(accessToken); //get this below  
var pp = new PasswordProfile() //required  
{  
  ForceChangePasswordNextLogin = true,  
  Password = "Watermelon1!"  
};  
var u = new User  
{  
  DisplayName = displayName,  
  UserPrincipalName = upn,  
  PasswordProfile = pp,  
  MailNickname = displayName.Replace(" ", string.Empty),  
  UsageLocation = "US",  
  AccountEnabled = true,  
  ImmutableId = Guid.NewGuid().ToString()  
};  
try  
{  
  var p = gc.Add(u);  
  Console.WriteLine("Created {0}, immutable ID: {1}", p.UserPrincipalName, p.ImmutableId);  
}  
catch (GraphException ex)  
{  
  Console.ForegroundColor = ConsoleColor.Red;  
  Console.WriteLine("{0}: {1}", ex.ErrorMessage, ex.ErrorResponse.Error.Message);  
}
```

Pretty straightforward…until you get to

`gc.Add(u);`

chances are you’ll blow up with a 403 Forbidden. In fact, chances are high, like 100% this will happen (if it doesn’t let me know).

#### Graph Read/Write

For whatever reason, and I’m still trying to figure out _exactly_ why, the ‘Read and write directory data’ permission doesn’t appear to allow adding users. I’m assuming this is because they want a user who’s in one of the principal management roles, like User Administrator (see [this post](http://jpd.ms/azure-admins-vs-azure-ad-admins/ "Azure Admins vs. Azure AD Admins") for some info on that), as opposed to allowing app principals to do this. The long and short is that the Graph API wants you to go through an OAuth browser flow to delegate a token from a user with the appropriate permissions. If you’re using ADALv2, there’s no

AcquireToken

overload that’ll do this. This is fine, unless you want to automate the creation of these users.

#### What are you to do?

Fortunately, we can use the OAuth password grant\_type to request a token with only a user’s username & password.

#### AccessTokens & the Graph

You’ll need a few things to get setup. I’m not going to go into much detail here, because if you’re encountering this issue chances are you’re already well setup. We need to request a token from the AAD STS, including both the user’s username/password, _as well as the client ID and secret_ of the app you’re developing. Here’s a sample:

```c#
var reqUri = "https://login.windows.net/YOUR TENANT ID OR NAME/oauth2/token";  
var postData = "resource=00000002-0000-0000-c000-000000000000&client\_id={0}&grant\_type=password&username=john%40mytenant.onmicrosoft.com&password=nicetry&scope=openid&client\_secret={1}";  
var wc = new WebClient();  
wc.Headers.Add("Content-Type", "application/x-www-form-urlencoded");  
var response = wc.UploadString(reqUri, "POST", string.Format(postData, AppId, EncodedKey));  
var tokenData = JObject.Parse(response);  
return tokenData\["access\_token"].Value();
```

Let’s deconstruct this request a bit, shall we?

#### [https://login.windows.net/YOUR](https://login.windows.net/YOUR) TENANT ID OR NAME/oauth2/token

This is where we’re posting our token request

#### PostData

The main chunk of our request.

* `resource` The resource you’re trying to access. In this case, it’s the graph, which always has this ID: `00000002–0000–0000-c000–000000000000`
* `client_id` your app’s client ID
* `client_secret` Important — make sure to URL encode your key before putting it here
* `grant_type` `password`
* `username` your UPN (e.g., blah@tenant.onmicrosoft.com)
* `password` this should be obvious
* `scope` Get data in the [OpenID Connect](http://openid.net/connect/) format: openid  
And make sure you URL encode the form/values before submitting, otherwise it’s 400s for you.

#### And that’s it

Provided you got a token back and didn’t have any problems with the request, you should be able to tack that access token into the header

Authorization: Bearer ...access token...

or you can stuff that into

`new GraphConnection(accessToken)`

if you’re using the Graph Client wrapper.

Create away! You’re off.

![srs](/img/0_8E4OQw_jaF_uJW3F.png)
