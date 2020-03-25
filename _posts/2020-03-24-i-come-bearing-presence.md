---
title: Creating a Teams presence publisher with Azure Functions, local and cloud
description: ''
date: '2020-03-24T15:04:11.4261648Z'
categories: ['azure', 'azure-ad', 'teams', 'identity', 'functions']
keywords: ['azure', 'teams', 'azure-ad', 'presence', 'msal', 'tokencache']
slug: /teams-presence-publisher
---

_Want to go straight to the code? [Here it is](https://github.com/jpda/i-come-bearing-presence){:target="_blank"}_

## Teams Presence

Presence info has been around a long time - we had it in Skype for Business and its predecessors - Lync, OCS, LCS, etc. There were even devices you could buy with lights indicating presence - super useful, especially in more 'open' offices to achieve some focus.

![embrava blynclight](img/2020-03-24-15-36-02.png)

The Embrava lights were popular, I had one on my desk, in fact - but they don't seem to work with Teams presence. Combine that with more recent news:

- [Presence is now in Microsoft Graph](https://developer.microsoft.com/en-us/graph/blogs/microsoft-graph-presence-apis-are-now-available-in-public-preview/){:target="_blank"}
- [Working from home is a new reality for many people](https://news.microsoft.com/covid-19-response/){:target="_blank"}
- [Working from home with an entire family](https://ed.sc.gov/districts-schools/schools/district-and-school-closures/){:target="_blank"}

And since I have little ones at home, it got me thinking about ways to let them know if I'm on the phone or not. I wanted something multitenant that could publish presence for anyone, not just me. In that spirit - any solution worth a solution is worth an over-engineered solution, right? _Right?!_ Click the video to see it in action.

[![video](https://img.youtube.com/vi/ujDyD63KdbA/0.jpg)](https://www.youtube.com/watch?v=ujDyD63KdbA){:target="_blank"}

At home I've got a few bulbs, in my office but also upstairs so my family can see if I'm on the phone or not.

![teams presence bulbs](img/2020-03-24-16-06-04.png)

Same for the office - rather than a small light, I felt the size of this lamp really reinforced the message.

![office presence](img/2020-03-24-16-06-45.png)

## Let's build!

Here's what we're going to build:

![runtime sketch 1](img/2020-03-24-16-40-00.png)

A presence-poller running in Azure Functions, which...

- logs in a user and stores a `refresh_token`,
- polls the Graph for Presence updates,
- stores the update (if any) in Azure Table Storage, and
- notifies subscribers over a Service Bus topic.

Plus, we have a local Azure Function, which...

- runs in a container on a raspberry pi,
- creates a subscription to the Service Bus topic, and
- interacts with the local Philips Hue Hub over HTTP

There is no webhook or push support yet for Presence updates, so until then, we need to poll for it ourselves. Rather than having multiple devices poll for the same data, I thought it prudent to poll from one component, then push to as many subscribers as are interested. The local Function acts as a shim for interacting with whatever local devices are interested in presence updates. In my case, two Philips Hue Hubs - one in the office, one at home. This also helps with firewall silliness - since the poller runs cloud side but publishes changes to a topic, our local function only needs outbound internet access - no inbound access required.

## Presence updater job

The [`presence-refresh`](https://github.com/jpda/i-come-bearing-presence/blob/2d633408772be906a9c1423eef31e8ec6a4447c2/ComeBearingPresence.Func/PresencePublisher.cs#L103){:target="_blank"} function calls `CheckAndUpdatePresence`, which does the bulk of the heavy lifting. This timer job is responsible for actually pinging the Graph to fetch a user's current presence, check it against the last known presence, and, if different, publish it to subscribers. I'm using the user's ID as the service bus topic name - this way we have enough information at runtime to know which topic to use for sending updates.

First, we need to know which user for which we want presence data. This is driven by a store of accounts in Table storage. The key here is _how_ the accounts get into the list in the first place - and how we get access tokens for querying the graph.

## Authenticating users and asking for consent

Of course, being the Graph, everything here is authenticated via Azure AD. We need the `Presence.Read` permission, which (as of March 2020) is an admin-consentable permission _only._ This means that only an administrator of an Azure AD tenant can consent to apps using this permission. There are some reasons we can infer from that - a malicious app that knows your presence could risk far more information exposure than your organization (or you) would be comfortable with. Once we have an app with that consent, we still need to capture individual user consent (you can, of course, consent tenant-wide as an administrator) and record that this user would like to get their presence info published to them.

To configure this, first we need to register an app. The app itself needs `Presence.Read` but little else. We'll also need certain reply URLs registered. You can read more about this process [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app){:target="_blank"}. You'll get a client ID (application ID) and secret (a random string, or a certificate). Save them off, we'll need them in a little bit. Note I'm using a certificate here for authentication, since this `Presence.Read` is a sensitive scope. We also need a way to securely share that secret (be it a string or a certificate) with our Function.

We'll use KeyVault and Managed Identity for accessing the secret. This way, our Function will have an identity usable only by itself - we'll assign this identity permissions in KeyVault to pull the secrets, so we don't keep any secrets in code. Alternatively, you can assign the Presence.Read permission directly to your Managed Identity, removing the need for KeyVault entirely. Read on [here](https://docs.microsoft.com/en-us/azure/key-vault/managed-identity){:target="_blank"} for more info on using KeyVault with Managed Identities.

Alternatively, if using certificate authentication to Azure AD, you can store the certificate in App Service directly, then reference it using normal Windows certificate store semantics (e.g., CurrentUser/My, etc). In fact, I had to switch to this for testing, as I was having issues getting connected to KeyVault reliably.

Next, let's get into the `auth-start` and `auth-end` functions, which actually authenticate our users.

### `auth-start`

This function is responsible for two things - authenticating a user to capture `access_` and `refresh_token`s, in addition to storing the user in the PresenceRequests table (e.g., please start polling for presence updates). We need the `Presence.Read` and `offline_access` scopes so we get a refresh token. Auth start doesn't do a whole lot except redirect the user over to Azure AD to authenticate. We're going to use the `authorization_code` flow here to keep any tokens out of the user's browser. After the user signs in, we'll ask Azure AD to do an HTTP POST back to us with the authorization code in tow. While we could generate that URL ourselves, we can hand it off to MSAL here using `ConfidentialClientApplication.GetAuthorizationRequestUrl`, making sure to also include `response_mode=form_post` to ensure we're keeping the code out of the URL.

### `auth-end`

This function handles the return trip from Azure AD. Notably, it receives the `authorization_code` Azure AD generates after a successful sign-in and authorization, sends that _back_ to Azure AD to receive an access_token & refresh_token for the requested scopes. We could, again, handle this ourselves, but better to let a library do the heavy lifting - MSAL's `AcquireTokenByAuthorizationCode` handles the interaction with Azure AD, then caches the tokens for us.

Next we store the fact that a user authorized us in tables - this is what the timer job uses to determine which presences to request.

## Token caching & MSAL

Rather than reimplement an entire token cache, since we're only interested in changing the _persistence_ of the cache and not the mechanisms of how the cache is serialized, MSAL provides us some delegates we can set - namely, `BeforeAccess` and `AfterAccess`. Set these delegates to your own methods to handle the last-step persistence (and rehydration) of your preferred storage medium. I'm using table storage here again, so my Before and After access delegates are only concerned with writing and reading the bytes to and from table storage.

Now - [if you read through the guidance from the MSAL team in the wiki & in the docs](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization#token-cache-for-a-web-app-confidential-client-application){:target="_blank"} - you'll notice this:

> A very important thing to remember is that for Web Apps and Web APIs, there should be one token cache per user (per account). You need to serialize the token cache for each account.

In web apps, this is fairly straightforward to accomplish - a user is signed in, so we usually have some bit of identifying information about them to use as a cache key. This way we can query _something_ like Redis, Cosmos, SQL, etc to fetch our MSAL cache. In our case, however, we don't have an interactive user except for the first time. In fact, we want to keep a very Microsoft-Flow-esque user experience: sign in, store off tokens, do background work as necessary. This means that when we run our timer job without any sort of user context available, it gets dicey trying to figure out which token cache to use.

`TokenCacheNotificationArgs` (what your BeforeAccess and AfterAccess delegates will receive) don't have enough user data for us to determine which cache to use, because we don't have an `Account` object yet - at timer job runtime, we literally have an empty MSAL object - configured, but no accounts or data yet. That MSAL's main entry point objects are called 'Applications' (`PublicClientApplication` and `ConfidentialClientApplication`) is a bit misleading. In reality, the uniqueness per application object is really the token cache itself. What I learned here was that, in fact, we don't want to use a singleton ConfidentialClientApplication, instead we want a `ConfidentialClientApplication` _per unique token cache_ which, per the guidance, is per-user. Beyond that, a singleton `ConfidentialClientApplication` would cause problems with cache serialization - as all accounts within the object at the time are serialized instead of a notion of a 'swappable' cache in the same object. Needless to say, I spent a ton of time trying to figure out the best way to go forward.

What I ended up with is an MSAL client factory. Not a big fan of it, but it lets me request an MSAL ConfidentialClientApplication configured with the right caches per user, sent in at runtime. Find it [here](https://github.com/jpda/i-come-bearing-presence/tree/master/ComeBearingPresence.Func/TokenCache){:target="_blank"}.

You may also notice a transfer between transient & actual keys. The reason for this is that during the `auth-end` callback, we have no context to know who the user is - so when we create our CCA to consume the code (`AcquireTokenByAuthorizationCode`), if we don't set the cache delegates upfront, they don't get persisted. This sets the cache before with a random identifier, which is then renamed in storage to the user's actual identifier once we know who the user is.

Rube Goldberg would be proud.

Once we've plowed through this circus, we have a reliable way to get access_tokens for calling the graph. At this point, we're publishing presence changes from the graph to a topic! Our cloud-side work is done.

## Publishing to subscribers

The Service Bus topics are created per user ID, each subscriber should have its own topic subscription (unless they are competing, in that they are doing the same work). I have two locations, Home & Office, so I have two subscriptions, one for each. If I had multiple _instances_ of a subscriber working on a common goal (e.g., updating the lights at my home), those instances would share the same subscription.

## Subscribing to updates and manipulating our Philips Hue Hub

![CNN: new pope smoke](img/2020-03-24-21-19-22.png)

OK! Now we've gotten our access token and we're polling the graph. Great! Now we need something to listen for those status changes. This could be anything you want - a light, a sign, or even a puff of smoke (like when they pick a new pope). I'm using Philips Hue bulbs, but my colleague [Matthijs](https://twitter.com/mahoekst/status/1215179713888391168?s=20) built some wild stuff with his presence and a bunch of boards and hundreds of LEDs, plus a really cool MSAL Device Code flow.

Talking to the Hue Hub means creating the equivalent of an API key on the local Hub, then using that in subsequent requests. I looked at Philips' online stuff, but you can only have one Hue hub per account? Seems like a strange limitation, but since I have one at home and at work, I figured that wasn't going to work out. Instead, I'll just talk to them locally.

Creating an api key to talk to the hub locally is a pretty quick one-time procedure. You can do it manually or automate it - but you'll need to press the button on your Hub, then run your code within a window of time. See more about creating Hue keys [here](https://developers.meethue.com/develop/get-started-2/){:target="_blank" rel="noreferrer"}.

Now on to our [LocalHueHubSubscriber](https://github.com/jpda/i-come-bearing-presence/tree/master/LocalHueHubSubscriber.Func){:target="_blank"} function. This function is what runs locally, with a network path to your Hue Hub. There is a dockerfile to build targeting `dotnet2.0-arm32v7`, which runs on a raspberry pi. This function is what's going to ping our Hue Hub with new colors depending on status. I have a crude status-to-color map (find that [here](https://github.com/jpda/i-come-bearing-presence/blob/2d633408772be906a9c1423eef31e8ec6a4447c2/LocalHueHubSubscriber.Func/Function1.cs#L25){:target="_blank"}) - it uses a Service Bus trigger and when it fires, makes an HTTP call to the hub with the desired color.

On your raspberry pi, all you need is docker. Use the `.env.local` file to store configuration and push it into the container at runtime. You'll need your SB topic subscription's connection string, plus your local Hue Hub's IP and path to your light group.

## todo

- UI for adding user endpoints
- ARM template for Azure resources
- finish code for creating SB topics when new user onboards

:bowtie:

Find me at [@AzureAndChill](https://twitter.com/AzureAndChill){:target="_blank"} with any questions or concerns!
