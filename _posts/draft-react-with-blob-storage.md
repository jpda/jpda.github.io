---
title: Creating a Teams presence publisher with Azure Functions, local and cloud
description: ''
date: '2020-03-24T15:04:11.4261648Z'
categories: ['azure', 'azure-ad', 'teams', 'identity', 'functions']
keywords: ['azure', 'teams', 'azure-ad', 'presence', 'msal', 'tokencache']
slug: /teams-presence-publisher
---

{% youtube yR9lf6aqmsE?t=1h11m11s %}

_[take me straight to the code!](#the-code)_

On Tuesday's stream, after our great talk with Simon Brown, we decided to dig into building a fully client-side app that connects to Azure Blob Storage.

## What is blob storage?

Just that - storage for blobs of data, big and small. Historically it stood for 'Binary Large OBjects' although that was mostly used in SQL circles for storing data in databases. Regardless of the origin,  blob storage (aka S3 at AWS) is a staple of modern apps. 

Azure Blob storage has some unique features that make designing apps even easier. For example - a [standard storage account](https://docs.microsoft.com/en-us/azure/storage/common/scalability-targets-standard-account) has a maximum egress of up to _50 Gb/s!_ - that's 50 Gb/s that your app server/platform doesn't have to handle on its own. 

This works for upload too - standard storage in the US has a max ingress of 10Gb/s. Clients uploading or downloading directory to and from storage accounts can have a _massive_ impact on your app's design, cost and scalability.

We've seen customers leverage this over the years - for example, streaming large media assets (think videos, pictures, datasets) from blob storage _directly_ to clients instead of proxying through your app server. 

Take this scenario - I want to share videos and pictures with people I work with, or with the internet as a whole. Previously, I would have had some storage - a network share, NAS device - and my app server would have some sort of API exposed to access that data. My app would have to send and receive data from clients, which meant my app servers would need enough bandwidth for pushing and pulling all that data around.

By using storage directly, my servers and APIs can direct clients to upload and download directly from storage, significantly reducing compute bandwidth requirements, with the benefit of a worldwide footprint of storage locations. 

## But how do we ensure secure access?

Historically, we used [shared access signatures (SAS)](https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview) tokens with storage, which are time- and operation-limited URLs with a signature for validation. For example - I'd like **Read** access to **`https://storageaccount.blob.core.windows.net/container/blob1.mp4`** for the next **60 seconds** - this would generate a URL with some parameters, which was then signed with the master storage account key, then the signature was tacked onto the end of the URL. Then we share that URL with whatever client needed to do the operations.

This was cool, except it meant we needed some server-side API or web server to store and manage the master account key, since can't send it directly to the client.

## Enter Azure AD & Storage Blob Data RBAC

If you're familiar with Azure, you know there are two distinct 'planes' - the control plane (the management interface) and the data plane (the actual resource data). I like to think of it as the difference between being able to deploy a VM vs actually having credentials to RDP or SSH into it.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/frkorhgfd60sta5zjtub.jpg) <figcaption>If you've seen this screen before, you've used Azure RBAC</figcaption>

All Azure resources have some degree of control plane role-based-access-control - things like 'Resource group owner' or 'resource group reader' - that allow management operations on those resources. Over time more and more data plane operations have been added, so we can use Azure RBAC for controlling both who can manage the resource as well as who has access to the resource or data itself. The advantage here is furthering the 'least privilege' mantra - a storage key is the key to the proverbial castle, so to speak, so if we can limit operations on an ongoing basis, we can limit the blast radius of any bad actors.

Storage has roles specifically for connecting to the account's data plane - connecting to blobs specifically, for example. In the IAM/role assignments blade for the storage account, note the 'Storage Blob Data...' roles. These give Azure AD accounts (users _and_ service principals) access to the blobs directly. 

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/rjwifo8x5mjzqtei7514.jpg) <figcaption>The different storage-related roles</figcaption>

We're going to use this to build our client-side blob reader app.

## Bill of Materials <a name="the-code"></a>

We're going to:

- deploy a storage account to Azure
- add a user to the `Storage Blob Data Reader` role
- Create a quick-and-dirty React app
- Add Azure Identity dependencies
- Register an app in Azure AD to represent our react app
- Authenticate the user and list out our blobs

First, create a blob storage account in Azure. General Purpose v2 is fine for what we're building. I use Locally-redundant storage (LRS) for my account, but pick what's best based on your requirements.

Next, we'll go to the IAM blade of your storage account. Here we need to add a role assignment of Storage Blob Data Reader to a user you're going to sign-in with. This could be yourself or a test account.