---
title: DocuSign + SharePoint Online
description: ''
date: '2014-07-17T02:33:20.000Z'
categories: []
keywords: []
slug: /docusign-sharepoint-online-c2bf29b8a207
---

Document signing + SharePoint Online with non-licensed users flew across my desk at WTFHQ today. Here’s the basic requirement:

> Licensed users need to store PDFs in SharePoint while getting them digitally signed by non-licensed SharePoint users. Preferably without requiring creating External Users in SharePoint.

I’m happy to report this works quite easily with DocuSign + SharePoint Online. You’ll get SharePoint doc lib integration e.g., the context menu will offer you options on documents like ‘Sign with DocuSign’ and ‘Get Signatures with DocuSign.’

We’ll start with the main case — a licensed SharePoint user needs to get a document signed by a non-SharePoint user. It’s _really easy._

#### It’s really pretty easy.

When you use the ‘Get Signatures with DocuSign’ option, you’re sent over to DocuSign, where you login (either with your Office 365 account or your DocuSign account), and it’s all vanilla from there — mark the required fields to collection from the target and drop their email addresses into the invitation field.

\[caption id=”attachment\_252" align=”alignnone” width=”705"\]

![1-get](https://cdn-images-1.medium.com/max/800/0*AkhkQ4VYm-J2nPR-.png)

Document library integration\[/caption\]

\[caption id=”attachment\_253" align=”alignnone” width=”922"\]

![Invite non-licensed users to sign](https://cdn-images-1.medium.com/max/800/0*KLhHDSCjJoivWTjQ.png)

Invite non-licensed users to sign\[/caption\]

The target gets an email, inviting them to sign the document. When they click the link, they’ll go directly to DocuSign to sign the document, no SharePoint account required. They get an option to download the document, and the signed copy goes back into your document library. For our scenario, we’re finished.

\[caption id=”attachment\_254" align=”alignnone” width=”609"\]

![Mail received by the target signer](https://cdn-images-1.medium.com/max/800/0*3tu2y7o0NIljtakm.png)

Mail received by the target signer\[/caption\]

\[caption id=”attachment\_255" align=”alignnone” width=”756"\]

![Signed document automatically back in SharePoint library](https://cdn-images-1.medium.com/max/800/0*2PW95kOOgm11YUiu.png)

Signed document automatically back in SharePoint library\[/caption\]