---
title: Denying Access through ADFS + Yammer
description: ''
date: '2014-06-10T03:17:49.000Z'
categories: []
keywords: []
slug: /denying-access-through-adfs-yammer-180667656530
---

_Start_ [_here_](http://jpd.ms/denying-access-to-adfs-secured-applications/ "Denying Access to ADFS-secured Applications") _if you haven’t already._

We’ll start with the last example — I’m piloting Yammer, I’ve got some users I want to grant access, but _a whole lot more_ I want to deny. In the case of Yammer and likely some other RPs who don’t understand the Permit/Deny claim, you’ll have to manipulate something else to force the RP to boot you out. In Yammer’s case, they use the email address as the SAML\_SUBJECT, which makes them pretty easy to poke.

Really, you should just update to ADFS 3.

But since that’s easier said than done, here’s how to make Yammer deny access to people using ADFS claims transformation rules.

#### Recursion? Did you mean recursion?

You’ll note from the previous post that we were denying users based on _extensionAttribute1._ This isn’t going to work any more, since Yammer doesn’t process the Deny claim, and punishes your insolence by stuffing you into an infinite redirect loop. The first thing you’ll want to do is remove any Issuance Authorization policies you have and put back ‘Allow all users.’

Next, we need to break users where _extensionAttribute1_ doesn’t equal false.

#### Persona Non Grata

In the case of Yammer, it’s easiest to just send in an invalid email address. Not invalid as in syntactically incorrect, but invalid for your organization.

That’ll give users a proper error message, informing them of their denial.

Two rules should do the trick (and you could probably get it down to a single composite rule) — one to transfer the email address to the SAML\_SUBJECT and one to overwrite that claim if the user doesn’t have the requisite attributes.

To overwrite the claim (as opposed to adding a _second_ value to the same claim), your issue statement should include the Issuer, OriginalIssuer and ValueType as the existing SAML\_SUBJECT claim.

Something like this:

> EXISTS(emailClaim:\[Type == “http://schemas.microsoft.com.../emailAddress"\]) && NOT EXISTS(c:\[Type == “http://schemas.jpd.ms/unique/ad/Authorized", Value == “true”\])  
> \=> issue(Type = “SAML\_SUBJECT”, Value = “FAKE@domain.com”, ValueType = emailClaim.ValueType, Issuer = emailClaim.Issuer, OriginalIssuer = emailClaim.OriginalIssuer);