---
title: "DocuSign + SharePoint Online"
date: 2014-07-17T02:33:20.000Z
author: "John Patrick Dandison"

---

Document signing + SharePoint Online with non-licensed users flew across my desk at WTFHQ today. Here’s the basic requirement:
> Licensed users need to store PDFs in SharePoint while getting them digitally signed by non-licensed SharePoint users. Preferably without requiring creating External Users in SharePoint.

I’m happy to report this works quite easily with DocuSign + SharePoint Online. You’ll get SharePoint doc lib integration e.g., the context menu will offer you options on documents like ‘Sign with DocuSign’ and ‘Get Signatures with DocuSign.’

We’ll start with the main case — a licensed SharePoint user needs to get a document signed by a non-SharePoint user. It’s _really easy._

#### It’s really pretty easy.

When you use the ‘Get Signatures with DocuSign’ option, you’re sent over to DocuSign, where you login (either with your Office 365 account or your DocuSign account), and it’s all vanilla from there — mark the required fields to collection from the target and drop their email addresses into the invitation field.

[caption id=”attachment_252&#34; align=”alignnone” width=”705&#34;]




![1-get](http://jpd.ms/wp-content/uploads/2014/07/1-get.png)



Document library integration[/caption]

[caption id=”attachment_253&#34; align=”alignnone” width=”922&#34;]




![Invite non-licensed users to sign](http://jpd.ms/wp-content/uploads/2014/07/1-invite.png)



Invite non-licensed users to sign[/caption]

The target gets an email, inviting them to sign the document. When they click the link, they’ll go directly to DocuSign to sign the document, no SharePoint account required. They get an option to download the document, and the signed copy goes back into your document library. For our scenario, we’re finished.

[caption id=”attachment_254&#34; align=”alignnone” width=”609&#34;]




![Mail received by the target signer](http://jpd.ms/wp-content/uploads/2014/07/2-mail.png)



Mail received by the target signer[/caption]

[caption id=”attachment_255&#34; align=”alignnone” width=”756&#34;]




![Signed document automatically back in SharePoint library](http://jpd.ms/wp-content/uploads/2014/07/3-signed.png)



Signed document automatically back in SharePoint library[/caption]
