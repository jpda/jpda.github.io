---
title: Retrofitting OIDC to legacy systems via reverse proxy
description: ''
date: '2019-10-30T08:24:18.1261648Z'
categories: ['azure', 'azure-ad', 'identity']
keywords: ['azure', 'coldfusion', 'oidc', 'apache']
slug: /oidc-retrofit
---

Obviously we want systems that use modern authentication. In the case of Azure AD, you get _the good stuff_, like [Conditional Access](https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/overview){:target="_blank"} and a whole host of new authentication mechanisms, like [Windows Hello](https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-overview){:target="_blank"} &amp; [FIDO2](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-passwordless){:target="_blank"}.

This is a big jump, however, especially for old apps. And by old, I mean _old_ - running on unsupported or deprecated platforms (or versions of those platforms), where the skillset doesn't exist in your circle to update it, or where the vendor who created the platform has ceased to exist. Consider too that some legacy systems may be on-deck for total replacement, either with custom dev or off-the-shelf systems, where we want to keep costs as low as possible.

Like most modernization projects, there are different ways to approach the problem - in this case, I'm focused on the 'moat-building' method, where we dig out and isolate the problem systems and proxy access through more modern systems. This is a bit quick-and-dirty, but for systems that are slated for replacement or just can't be touched, it's a low-impact way to wrap new functionality around an old problem. You can apply this to all sorts of parts of a system as well - e.g., extracting and shipping data out of an old system into a new system and schema while slowly moving referencing bits over to the new system.

Let's dig in.

## Reverse proxying

Since we've got a web app and we want to add only authentication, it's relatively straightforward. We need to:

- Authenticate the user, using a typical oidc-tango
- Redirect to identity provider
- Consume and validate issued token
- Read claim data
- Transform some claim data before forwarding along
- In our specific case, we need specific values from the claims to be forwarded in a specific header

## Layout

![layout](img/apache-jwt-00.png "layout")

## Considerations

Since we're usurping the user path to the app, we'll need to make sure we manipulate the network environment &amp; DNS in a way to make the app either inaccessible or unusable if someone was to hit it directly. If hosted in Azure, this could be something like a two-subnet VNet, one subnet with the app and the other with the proxy, with NSGs locking down the app subnet to only allow traffic from the proxy subnet. On-prem there are myraid ways to segment networks and restrict access. 

You'll also want to host with TLS, Azure AD reply URLs will require `https` except in the case of localhost. 

## Apache config

For apache, we're going to use [`mod_auth_oidc`](https://github.com/zmartzone/mod_auth_openidc){:target="_blank"} which is an [OIDC-compliant](https://openid.net/certification/){:target="_blank"} relying party/client module for OpenID Connect.

Let's take a look at the config.

{% gist 3417e90338485374332d3e857cd7dd61 %}

## Results

The `mod_auth_oidc` package includes all the claims as passthrough headers, in addition to our custom header with our transformed value. My source claim in this case was `preferred_username`, which we transformed via apache to `X-jpda-header-loc`.

The advantage to this method is potentially _many_ apps could live behind this proxy, with very little additional effort to onboard more. Of course the tradeoff with proxying is a single choke point for traffic, so carefully consider which apps should be grouped behind specific instances.

![claims](img/apache-jwt-00.png "claims")
