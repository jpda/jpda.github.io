---
title: Denying Access to ADFS-secured Applications
description: ''
date: '2014-06-10T01:59:32.000Z'
categories: []
keywords: []
slug: /denying-access-to-adfs-secured-applications-1a897de3876b
---

I’m going to have to make this a two-parter, because _some company \*ahem\*_ Yammer — doesn’t appear to handle the Deny (http://schemas.microsoft.com/authorization/claims/deny) claim very well. By very well, I mean _at all._

Here’s the scenario — you’re piloting an application, likely a cloud-based or service-based application, and it’s using ADFS to authenticate (think Office 365, Yammer, Salesforce, etc). The key word here is _pilot _— you have some users you want to deny access.

But let’s back up a second — ADFS is, in my opinion, proper for _authentication,_ but not authorization. ADFS is a means to validate your identity, but not a means to grant access to resources. That’s true in the purest of forms, but when an application doesn’t offer a valid way to deauthorize users, sometimes it’s easier to go to the source.

In the case of Yammer, restricting users is _painfully bad — _particularly for an enterprise app. Microsoft bought Yammer almost two years ago, so we should hope that things will get better.

#### Let’s talk claim rules.

Claim rules let you do all kinds of fun stuff — from manipulating claims before they’re sent to the relying parties to even determining if that user is authorized to access that relying party (by not sending any claims, which effectively denies them access).

Unfortunately, claim rules _also_ use some regular expressions, which make my eyes bleed. But no matter, we must press on.

#### A simple example.

Let’s start simple. I want to toggle access to a specific ADFS application using a value from an AD attribute on a user. For this example, I’ll be denying myself access to the Microsoft Identity platform (Office 365, Azure, MPN, etc — what could go wrong, right?), based on a value in extensionAttribute1 in my AD profile. Of course, if you are a _real_ pro and have extended your schema in AD, you can, of course, use your attribute.

Anyway, so I’ve got this going on:

*   Relying Party: Microsoft Office 365 Identity Platform (this is what you setup to federate with O365)
*   AD Attribute: extensionAttribute1, value ‘false’

Everything else is vanilla ADFS.

For this we’re only using Issuance Authorization policies, but this same claim rule syntax is valid for other rule types as well.

#### Let’s add a new claim.

First thing — let’s create a new claim that we can use to stuff our value in. This isn’t absolutely required, but makes it a lot easier — and hey, maybe you’ll even use that claim value in your relying parties one day!

Here’s mine.

![claim](https://cdn-images-1.medium.com/max/800/0*2a9RJuUTXf6JGz5X.png)

#### To the relying party!

Now find your relying party. In my case, it’s Office 365. Let’s create some claim rules. Head over to your RP, right-click and edit your rules.

Here’s an important piece — **claims are really only in scope within the tab they’re created on** — for instance, if you’re in the Issuance Transform Rules tab, any custom rules you create (for instance, to assign an AD attribute’s value to a claim) are only valid within the scope of that tab. There may be some exceptions to that rule, but for the most part that’s how it’s compartmentalized. If you think about what you’re doing here, it makes sense — Issuance policies are different from transformation policies, etc — but I fell into this trap so hopefully you won’t have to.

In my example, I want to deny access based on a user account’s _extensionAttribute1_ being set to false.

First, we need to get a value into our fresh claim. You’ll need to do a custom rule, but it’s pretty simple:

> c:\[Type == “http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == “AD AUTHORITY”\]  
> \=> issue(store = “Active Directory”, types = (“http://schemas.jpd.ms/unique/ad/Authorized"), query = “;extensionAttribute1;{0}”, param = c.Value);

Let’s dissect that a bit. No regular expressions! There’s a quick win.

We’re going to use AD to populate our new claim (http://schemas.jpd.ms/unique/ad/Authorized) from our _extensionAttribute1_ AD property. Simple right?

Since rules are processed in order by the rules engine, this rule needs to come first. Now our subsequent rules can use the value of that claim (Authorized) to make decisions on token issuance.

Here’s my next rule:

> c:\[Type == “http://schemas.jpd.ms/unique/ad/Authorized", Value =~ “^(?i)true$”\]  
> \=> issue(Type = “http://schemas.microsoft.com/authorization/claims/permit", Value = “PermitUsersWithClaim”);

This one is so easy though that you can ‘cheat’ — just use the Permit or Deny based on Claim Value template — pick your claim (in my case, Authorized), set the value it should be equal to and you’re done.

In ADFS 3, you don’t even get sent to the relying party, you just get shut down at ADFS. I suspect this change is to support RPs that don’t know how to process the authorization claim.

Here’s my goofy ADFS login screen when my _extensionAttribute1_ is set to false:

![denied](https://cdn-images-1.medium.com/max/800/0*_i5P3ZeodcQHsRga.png)

You can, of course, get _crazy_ with your rules, but remember, you’re going to AD for this — and these rules don’t appear to be terribly ‘optimized’ in the sense that AD queries aren’t batched or anything. If you’re a high-volume identity shop, make sure your farm is well equipped to handle the extra load you can _possibly_ put on your infra with complex rules. Next up — dealing with RPs that don’t understand _Deny._