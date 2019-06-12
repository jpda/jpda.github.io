---
title: SharePoint 2010 RC User Profile Struggles
description: ''
date: '2010-02-26T03:43:00.000Z'
categories: []
keywords: []
slug: /sharepoint-2010-rc-user-profile-struggles-32f1ceb2324b
---

I finally got around to installing the RC of SharePoint 2010 yesterday, which spilled over into today. There were a lot of issues surrounding the User Profile Sync service in the beta 2, so much so that extremely detailed walkthrough has been posted on MSDN and other various sites about it.

Following those instructions still left me nowhere. The Event Log was blowing up with errors from the FIMSynchronizationService, telling me very little. The ULS Logs were also pretty unhelpful.

So…after killing the VM and starting fresh a couple times, I finally decided to get digging.

Here’s the setup:

*   Windows Server 2008 R2 Standard
*   SharePoint 2010 RC
*   Single service account with Replicate Domain Changes & Read access to the domain
*   All Service Applications provisioned and installed.
*   User Profile Synchronization service provisioned (i.e., ForeFront services started via Services on Server).

First, the Forefront Identity Manager is a new face to Microsoft Identity Integration Server (MIIS), which I was already mildly familiar with. By mildly, I mean totally unfamiliar and dangerous enough to do some REAL damage. I went looking in the event logs and other places, but it was a stonewall. I even tried running the miisclient.exe out of the %programfiles%\\Microsoft Office Servers\\14.0\\Synchronization Service\\UIShell\\ folder. That would give me a ‘Forefront cannot connect to server’ error.

That seemed easy to fix, I assumed, so I went looking around. Found the FIMOperators group and added my service account into that. Logged off, logged back on and BOOM — I was in. The MIISClient executable looks a lot like the old MIIS client, so I got to work.

I found three management agents:

*   ILMMA — FIM Service Management Agent
*   MOSS-User-Profile-Service-Application — Extensible Connectivity
*   MOSSAD-<name of your AD connection>

I tried to run the MOSSAD agent manually. This gave me a much more useful answer:

**failed-search — Replication access denied.**

I went into the run profile for the MOSSAD agent and went into the DS\_FULLIMPORT profile, since this was the one failing. I found that there were two steps — both of which were connecting to this strange AD partition:

CN=Configuration,CN=domain,CN=com

Being as my knowledge of AD is inadequate, I thought that perhaps there was some AD partition called configuration.

I did some searching, which led me [here](http://blog.jussipalo.com/2010/02/sp2010-fimsynchronizationservice-errors.html). I modified the steps I found on [Palo’s blog](http://blog.jussipalo.com), which you can find below. Palo suggests deleting the steps which pointed to the configuration partition, while I just changed my steps to point to my real domain. I’d be sure to check out his post first, and proceed with caution.

Back in Forefront Manager, I went to edit each of the steps, and both were pointing to the Configuration partition. In the drop down was a much more normal looking domain partiton:

CN=domain,CN=com

I selected that one for both steps, went back to SharePoint and kicked off a full synchronization. Sure enough, about 45 minutes later, we had our user profiles imported from the directory. After that I went through the rest of the run profiles and did the same.

My disclaimer here is twofold:

I don’t know what the Configuration partition is in AD. It may be important. I was working around this just to get my profiles imported. Since we’re not exporting any profiles at the moment, being a dev box and all, I took a leap and hoped for the best. This is probably not best practice. If I can find any detailed info on the configuration partition, I’ll post it.

That being said, be careful dealing with MIIS and AD. It is complex and powerful, and there’s a lot of potential for screwing things up. I’d suggest a test domain…strongly.