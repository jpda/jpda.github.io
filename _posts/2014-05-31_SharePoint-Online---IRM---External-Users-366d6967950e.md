---
title: SharePoint Online + IRM + External Users
description: ''
date: '2014-05-31T01:48:25.000Z'
categories: []
keywords: []
slug: /@jpda/sharepoint-online-irm-external-users-366d6967950e
---

Since I can’t seem to find _anything_ online regarding external users + IRM secured lists, I decided I should put it up here. In short,

> _External users using Microsoft Accounts can’t use IRM-secured documents that use an external client (e.g., Foxit)._

There are some nuances, however. Some scenarios work, some don’t. I did all of this testing from a fresh, non-domain joined Windows 8.1.1 VM.

#### Scenario I: IRM Office docs + External Users

This appears to work. I shared an IRM lib with a Microsoft account and got to work. I could open and view the documents (Excel, Word & PowerPoint) in the Office Web Apps and the IRM restrictions persisted.

#### Scenario II: IRM PDF + External User + Foxit Reader

For managed PDFs, it’s not nearly as straightforward. Managed PDFs require one of two readers, Foxit or NitroPDF. I only tried Foxit, because Nitro wanted money. First, managed PDFs don’t open in the Word Web App (like they used to, hopefully that will come back one day), they require a client.

I tried to open the PDF from SharePoint, which prompted a download & open. Upon opening, Foxit told me I needed the AD RMS connector, which is a free download. Downloaded & installed that, tried again, then I needed the Microsoft Online Sign-In Assistant (MOSSIA) — _another_ download/install. Did that.

The next time I opened Foxit, I was prompted by MOSSIA to sign in. Since the site was shared with my Microsoft account, I tried that. No dice — it just kept on kicking out my credentials. I tried app passwords, different Microsoft accounts, nothing.

I thought, perhaps it’s just broken, let me try the organizational account _that belongs to the tenant which owns the SharePoint Online instance._ This at least allowed me to login successfully, only to have Foxit kick me back out saying I didn’t have permission.

I killed Foxit and tried again — but now, my login information seemed to have persisted (granted, it’s what MOSSIA is supposed to do), so I was never prompted to login again. Fine, except that I couldn’t test any other accounts. Uninstalling MOSSIA didn’t help, so I’m guessing I need to whack some registry entries or some straggler files that are persisting my login information.

#### Scenario III/IV: IRM + External User (MSFT or Org) + Office 2013 client

I didn’t test this. It’s on my list, but I haven’t tried yet.

#### Scenario V: IRM PDF + External _Organizational_ User + Foxit PDF

Also haven’t tried this yet. I’ll be _really_ curious, but since the client I’m designing this for isn’t going to have external users with org accounts, it fell off the priority stack today.

#### A pseudo-solution

Since my specific parameters are IRM, PDF & External Microsoft account users, I’m left in a bind — there’s not a good story here. My parameters also are that the documents are read-only, so I found that if I convert the PDF to a Word doc and upload to the IRM-protected library, I can see it through the Word web app. That may not work for you, but it’s something to consider. It’s possible you could convert to some sort of an image as well, depending on your situation.