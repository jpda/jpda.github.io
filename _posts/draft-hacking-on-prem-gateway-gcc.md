---
title: Hacking on the On-prem data gateway for GCC-mid accounts
description: ''
date: '2021-02-10T21:09:08.4261648Z'
categories: ['azure-government', 'azure', 'power-bi', 'data-gateway', 'hacks']
keywords: ['azure-government', 'azure', 'power-bi', 'data-gateway', 'hacks', 'gcc', 'azure government']
slug: /gcc-mid-on-prem-data-gateway
---

I had an interesting one come across my desk here at WTF HQ earlier this week - before I knew it, I was up to my neck in decompiled data gateway code. Here's my adventure with fellow spelunker [@vstaropoli](https://twitter.com/vstaropoli).

## A cloud for every user, on every device, running Microsoft software

In the 70s, in the nascent days of Microsoft, Bill Gates said:

> A computer on every desk, and in every home, running Microsoft software.

A quote that lives on in spirit today. Only now we're talking about clouds. There are lots of clouds at Microsoft - Office, Azure, Dynamics, plus lots of variants for things like the US Government, China, etc. Today we're going to explore a bit of the terminology you'll hear, but more importantly, share a hack to unblock you if you're using Azure commercial cloud with on-prem data gateway but may also be using GCC Mid.

## Azure AD, Azure, Office, GCC and Government

A lot of new terms to define - things that are _related_ but not the same. Note that this isn't the exhaustive list, as there are still others (like Azure China).

- Azure AD Commercial/Global/Public: the main identity system that our clouds trust. This is what you're most likely using if you're not in China and not the US Government.
- Azure AD Government: the identity system that Azure Government and Office GCC High trust. These use totally different endpoints - `login.microsoftonline.us` for example.
- GCC Mid: this is a Microsoft offering for governments that want to use Office 365. Notably (and important for this discussion), this uses Azure AD Commercial.
- GCC High: this is a similar offering, but for government customers that require different certifications. This uses Azure AD Government.
- Azure Commercial: the main public Azure that you can go sign-up for today.
- Azure Government: a totally separate set of Azure services that meet specific US Government regulations.

Power BI also has different flavors, for different clouds. You can read more about those [here](https://docs.microsoft.com/en-us/power-bi/admin/service-govus-overview). Specifically, Power BI has a GCC version (aka GCC mid).

![Cloud to identity provider relationships](img/00-relationships.png)

So many terms with very similar names - Azure, Azure AD, Azure AD Government, Azure Government - it gets confusing very quickly, so precision is critical. Today we're going to explore the [on-prem data gateway](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-onprem), which was historically used by Power BI for accessing data sources the Power BI service couldn't access directly - like a database server running in your datacenter or an Azure VM you didn't want to open up to the internet. Think of it like a relay, with an agent installed somewhere that has a network path to the target servers. Over time, the scope of the tool has changed to add more services. You can use the OPDG for connecting things like Power Apps, Logic Apps and Azure Analysis Services to remote data.

Sadly, this is where things start to break down for GCC Mid users. Since GCC mid users are using Azure AD Commercial, they also use Azure Commercial for Azure resources (like Analysis Services). Unfortunately, when you type in a GCC-mid account, you lose any ability to connect to Azure Commercial - today, the guidance is to use ETL to move data between enviroinments, or use SSAS in a VM. But I think we can do better 😈

## Remote configuration? Ripe for ~~hacking~~ fixing

Since my buddy had called me about this nearly a dozen times, I figured it was time to roll up the sleeves and get to it - so we cracked open the configuration tool and started poking around. There was an assumption that the installer tool must be doing _something_ as we could login with a commercial account and everything was fine, same with a GCC High or Government account - there was something specifically happening in the installer for only GCC-Mid users. We started by decompiling the configuration tool, which was luckily dotnet. This made it easy to use tools like [ILSpy](https://github.com/icsharpcode/ILSpy) or [JetBrains' dotPeek](https://www.jetbrains.com/decompiler/) to take a look at the code and see just what was happening.

> Note
> I will never _ever_ use or suggest red-gate after what they did to .net reflector, and I'd suggest you don't either

Turns out, there _is_ some logic happening in the installer, specifically for setting environments! This is promising - it means that there must be _some_ way to override, or at a minimum, start man-in-the-middling this traffic to tweak to our whims. A bit more digging and we stumbled upon a configuration service - a _remote_ configuration service - which means it's ready to be fixed. It's actually a fairly straightforward issue - in essence, when this discovery service looks at a user's user principal name, it sends Power BI (or other services, like the On-prem data gateway) back a payload of account information - notably, endpoints. Let's take a walk through a couple of different scenarios.

### Normal, commercial user

Here's a normal user, someone who uses Office, Power BI and Azure Commercial. No government user here. The values are as we'd expect for a normal user - all commercial/public endpoints.

```json
{
    "azureActiveDirectoryUri": "https://login.microsoftonline.com", (Azure AD Commercial)
    "azureResourceManagerUri": "https://management.azure.com", (Azure Commercial - Resource Manager)
    "azureResourceManagerAadResource": "https://management.core.windows.net", (AAD Commercial configuration)
    "azureServiceManagementUri": "https://management.core.windows.net", (Azure Commercial - SM API)
    "microsoftGraphUri": "https://graph.microsoft.com", (Microsoft Graph commercial)
    "powerBiUri": "https://api.powerbi.com", (Power BI Commercial)
    "powerBiAadResource": "https://analysis.windows.net/powerbi/api", (AAD Commercial configuration)
    "azureStorageUriSuffix": "core.windows.net", (Azure Commercial - storage)
}
```

### GCC High (or DoD) user

Here's one for a GCC-High or DoD user - note _all_ of the endpoints are Azure Government related. `usgovcloudapi.net` and `*.us` is a sure giveaway here, although there are plenty of docs for cross-referencing Govcloud endpoints.

```json
{
    "azureActiveDirectoryUri": "https://login.microsoftonline.us", (Azure AD Government - ✔️)
    "azureResourceManagerUri": "https://management.usgovcloudapi.net", (Azure Government - ✔️)
    "azureResourceManagerAadResource": "https://management.core.usgovcloudapi.net", (Azure Government - ✔️)
    "azureServiceManagementUri": "https://management.core.usgovcloudapi.net", (Azure AD Government config - ✔️)
    "microsoftGraphUri": "https://graph.microsoft.us", (Microsoft Graph Government - ✔️)
    "powerBiUri": "https://api.high.powerbigov.us", (Power BI GCC High - ✔️)
    "powerBiAadResource": "https://high.analysis.usgovcloudapi.net/powerbi/api", (Azure AD Government config - ✔️)
    "azureStorageUriSuffix": "core.usgovcloudapi.net", (Azure Government storage - ✔️)
}
```

### GCC (aka GCC-Mid) user

And lastly, here's the main one we're concerned about: GCC (or GCC-Mid). Remember from earlier - a GCC user uses Azure AD Commercial _and_ Azure Commercial, not Azure AD Government or Azure Government. But look at what gets returned for a GCC-Mid user:

```json
{
    "azureActiveDirectoryUri": "https://login.microsoftonline.com", (AAD Commercial - ✔️)
    "azureResourceManagerUri": "https://management.usgovcloudapi.net", (Azure Government - ❌)
    "azureResourceManagerAadResource": "https://management.core.usgovcloudapi.net", (Azure Government - ❌)
    "azureServiceManagementUri": "https://management.core.usgovcloudapi.net", (Azure Government - ❌)
    "microsoftGraphUri": "https://graph.microsoft.com", (Microsoft Graph Commercial - ✔️)
    "powerBiUri": "https://api.powerbigov.us", (Power BI GCC - ✔️)
    "powerBiAadResource": "https://analysis.usgovcloudapi.net/powerbi/api",
    "azureStorageUriSuffix": "core.usgovcloudapi.net", (Azure Government - ❌)
}
```

Looks like we've found the culprit! Remember:

- GCC High and higher (DoD, etc) users use Azure/AD Government for everything
- Office & Azure Commercial use commercial for everything, BUT
- GCC-Mid users use AAD Commercial, Azure Commercial _and_ Power BI GCC

### Our new payload

Which means that for a GCC-Mid user, this API is mistakenly sending users to an all-government environment even though there are cases where a 'government' customer (e.g., GCC-Mid) is using Azure Commercial endpoints. If we look through this, we can cobble together our own, correct 'environment:'

```json
{
    "azureActiveDirectoryUri": "https://login.microsoftonline.com", (AAD Commercial - ✔️)
    "azureResourceManagerUri": "https://management.azure.com", (Azure Commercial - ✔️)
    "azureResourceManagerAadResource": "https://management.core.windows.net", (AAD Commercial configuration - ✔️)
    "azureServiceManagementUri": "https://management.core.windows.net", (Azure Commercial - ✔️)
    "microsoftGraphUri": "https://graph.microsoft.com", (Microsoft Graph Commercial - ✔️)
    "powerBiUri": "https://api.powerbigov.us", (Power BI GCC - ✔️),
    "powerBiAadResource": "https://analysis.windows.net/powerbi/api", (AAD Commercial configuration - ✔️)
    "azureStorageUriSuffix": "core.windows.net", (Azure Commercial - ✔️)
}
```

### Now we need to deliver this to the gateway during installation

This is probably the uglier part of this whole thing - now that we have a hypothesis of what we should send to the gateway, how do we intercept that call to inject our own stuff? Stuff like local DNS hijacking would work here, but since we really only need to tweak a single request it seems like a bit of overkill. Instead, since it's only during setup that we need to deal with this (and setup is already interactive), we can do it interactively with an HTTPS proxy. I personally like [Fiddler4](https://www.telerik.com/download/fiddler), with HTTPS decrypting turned on, plus the auto-responder. It makes this much easier to deal with.

First, make sure Decrypt HTTPS traffic is turned on (Tools --> Options --> HTTPS tab). Once you check this box, you'll need to add Fiddler's certificate to the trust store, which Fiddler will prompt you to do; remember that at this point, anything flowing through Fiddler will show a valid cert, since fiddler is terminating and re-wrapping the traffic with its own cert, which is now in your trusted roots.

![Fiddler https configuration](img/01-fiddler-https.png)

Next we want to use the auto-responder. This makes it super easy to let fiddler respond to certain requests that match a pattern. One easy way to do this is to let it go once, find the API call in question, then use that response as the base we're going to tweak.

To start, run the OPDG installer and at config time, make sure Fiddler is running. You can make it easier to find relevant traffic by using the bullseye and dragging it to the OPDG configuration tool, but not required. You're going to want to look for something calling `api.powerbi.com` - this is the request we want to fiddle with.

Once you've gotten one, I find it easier to copy the entire response out of the 'Raw' tab - this makes it easier to use later in our autoresponder rule. Copy out the raw body and drop it in notepad or similar. Next we want to create an autoresponder rule. There are lots of different ways to create rules, but luckily we have a relatively specific request we are looking for that, seemingly, isn't used elsewhere. Our URL matching rule then is quite simple:

- Add Rule
- `https://api.powerbi.com//powerbi/globalservice/v202003/environments/discover?user=`
- Create new response...

Good idea to click 'test' here to make sure your URL rule will run based on the URL.

![fiddler auto-responder rules](img/03-fiddler.png)

When you click save, the 'Create new response...' window will open, allowing you to drop in your new, modified response. Here's where we want to take the earlier response and update it to use the correct Azure Commercial-related URLs:

![fiddler auto-responder rules](img/04-new-response.png)

Note there is a lot of extra stuff in the response that stays largely untouched. We only need to modify the items in 'Our new payload' above and keep the rest the same. Once you've done that, click Save. 

Save this rule, make sure fiddler is on and capturing (bottom left corner and/or F12) and let's go back to the configuration tool. In here, it may have failed or it may have succeeded from earlier - you want to close and restart the configuration tool anyway, so go ahead and close and re-open. Make sure Fiddler is running before you re-start the configuration tool.

Once you type in your email address, you _should_ see your rule get hit - at which point, configuration will continue as normal. At this point, you can turn off/close Fiddler, as we're done with it. Now that the gateway is configured to use the Azure Commercial endpoints, the configuration process continues on normally, as if you were a commercial user. When you go into the portal to create the cloud gateway component, you should see your OPDG we just created as an option during resource creation.

While this is a bit of an ugly hack, it does enable you to start using the on-prem data gateway with Azure Analysis Services and Logic/Power Apps in Azure if you're a GCC mid customer. No idea as to the long-term supportability of this, but we've raised the issue and hope to see this built into the logic in the future. Happy hacking! 👍

Find me at [@AzureAndChill](https://twitter.com/AzureAndChill){:target="_blank"}
