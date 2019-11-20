---
title: Automating non-RBAC AKS and kubectl with Azure AD service principals
description: ''
date: '2019-11-19T20:08:17.5261648Z'
categories: ['azure', 'azure-ad', 'identity']
keywords: ['azure', 'aks', 'kubectl', 'service principals']
slug: /automating-aks-kubectl-with-service-principals
---

This came across my desk this morning and it happens frequently enough that I figured it worth writing about (write once, refer frequently!). Cloud services are made for automation from the ground up - automation is how you reach maximum efficiency of both the resources you create in addition to the people creating &amp; managing them. Of course, with automation, we still need security around who/what can login and manipulate those resources. Today we're going to look at using the Azure CLI with service principals &amp; `kubectl`.

## Why not user accounts?

If we take a trip back in time, when people _gasp!_ deployed and managed servers in their own datacenters, we'd create accounts in Active Directory or wherever and use them as service accounts. These service accounts were typically treated differently (e.g., with different policies, or different management attitudes) and used for servers, services and applications to get access to other resources. Your SQL Server might have its own domain account, your IIS appPools would run under a domain account, all sorts of things. But until late in the 00s, with the introduction of managed service accounts, these 'service accounts' were largely just 'accounts' that were used for services. They were not really any different from the account you or I might use to login to our PCs.

Once we transitioned to Azure AD, these types of habits continued.

> **SCENE** A non-descript office building somewhere in the world.  
> _(USER sits at large table. As USER sits, TABLE reveals itself to have a large touchscreen built in. "SUR40" flashes across the screen.)_  
> **USER**. OK, apps team needs a 'service account' for deploying their app in Azure..., hmm...  
> _(USER flips through stack of post-it notes. We see various URLs, like manage.windowsazure.com, portal.azure.com, aad.portal.azure.com, account.activedirectory.windowsazure.com, etc.)_  
> **USER**. Ah! Here it is.  
> _(USER opens web browser on TABLE. A Silverlight logo briefly appears, followed by a bugcheck. USER, defeated, pulls Surface Laptop out of bag and places it on TABLE.)_  
> **USER**. I need to get the details of this ticket out of my email.  
> _(After a lengthy pause, Outlook opens, then quits. USER grumbles indecipherably about Access, Exchange and JET databases before deciding to go straight to the ticketing system.)_  
> **USER** (Calling to someone off stage). What's our TF Service URL again?  
> **VOICE OFF STAGE**. I'll send it to you on our Teams team. It's contoso.visualstudio.com. Make sure you build an ARM template for anything you need to deploy. It's called azure devops now by the way...  
> **USER**. Thanks. I don't have vscode on my laptop so I'll have to use VS Online for this. OK, new service account - let's go to Users, Add New..  
> **USER** (to self). It's called Azure **Active Directory**, so surely it must be the same, right? Microsoft wouldn't give multiple disparate things the same name or rename the same thing every six months, right?  

Using user accounts for automation is a Bad Idea&#x2122; and we should strongly consider otherwise. There are many reasons, but here are the first ones that come to mind:

- Process - how do users on/offboard at your company? How frequently are passwords changed? Do you want to be on the hook for updating _n_ services every time you need a password change or reset?
- Availability - similar to above, what happens when your password expires and you're on vacation? Do all of your services go offline?
- Modern authentication pipelines &amp; experiences - things like MFA, conditional access, security keys, passwordless - all of this goes out the door if you're using a username &amp; password non-interactively.
- Security - what kind of services are available to your user account? What kind of lateral movement would an attacker be able to execute with your credentials?

Many people have written many words on this topic, so I'll move on - but the message is clear - _do not use user accounts for automation!_

## Service Principals

So what should we use? Azure AD offers Service Principals - these are effectively 'service accounts,' but we have far more control over both the scope of privileges &amp; access granted to the principal, in addition to being able to tightly control both credential and lifecycle. For example - Azure AD SPs can use passwords _or_ certificates for authentication. An organization with a centrally-managed certificate authority can rotate certificates on a schedule, keeping credentials entirely available yet out of the hands of developers. Beyond that, Managed Service Identity offers managed service principals tied to a resource (very much like managed service accounts from AD) where credentials are completely managed by Azure, but the service principal can be assigned permissions &amp; rights just like any other principal.

## RBAC vs non-RBAC AKS clusters

There are two ways to use AKS clusters in Azure - with or without Azure AD integration, usually referred to as 'RBAC-enabled' in most of the docs. The key difference here is related to the management vs. data planes of the Azure resource, in this case the AKS cluster. For example, think about an Azure VM. I may have _management_ rights to deploy/restart/change the VM's Azure configuration (disks, networking, resource group, etc), but may not have an account to RDP to the VM and make any changes there (the _data_ surface). Similarly, I may have the ability to _manage_ a storage account or SQL DB - change properties, move to different groups, add data sync partner regions, etc - but I may not have access to the data within those resources.

In a non-RBAC cluster, we have _management_ rights within the Azure portal to manage the resource, including getting credentials for kubectl. The difference between this route and the RBAC route is the type of credential - in a non-RBAC scenario, I'm using a service principal to fetch/generate a credential for AKS, but the credential itself is disconnected from the service principal. It is merely access to _generate a credential_ for the management interface. In an RBAC or Azure AD-enlightened cluster, not only will my service principal potentially be used to _manage_ the cluster, but I can also connect to the cluster _as the service principal_ which offers a greater level of granularity to the types of management operations allowed _within the cluster itself_, e.g., roles that are more granular than what is populated within the Azure management surface.

For an RBAC cluster, check the docs [here](https://docs.microsoft.com/en-us/azure/aks/azure-ad-rbac){:target=_blank} instead.

In our specific scenario, let's look at how we can create a service principal for use with a non-RBAC-enabled aks cluster &amp; kubectl. This is fetching credentials, but _not_ connecting _as the service principal_.

## Creating the service principal

This is pretty straightforward, if you have permission within your Azure AD tenant. In most cases you will. If you don't, you'll have to talk to your admin :unamused: Get logged into your `az` cli:

`az login`

Once we're logged in, hit it with this (full documentation with a ton of options for creating these is [here](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest){:target="_blank"}):

`az ad sp create-for-rbac --name "my-aks-automation-sp-name"`

Note that this SP is different than the one you configured when you created your aks cluster. This is a general purpose SP that you can use for all sorts of things (although I'd suggest using just one per task, with granular permissions). They're free, so create as many as you want.

The output of `az ad sp create-for-rbac` will give you a chunk of info - notably, your new service principal's ID (shown as `appId` in the response). If you go the password route (e.g., you didn't include a certificate), you'll also get a `password` and the name of the Azure AD tenant the SP was created within.

## Hooking our SP up with some permissions

Next we want to grant our SP some rights to resources. You can do this in the portal or CLI. In the portal, go to your resource - in our case, your AKS managed cluster. You only have to assign the rights once so no one will judge you for doing it from the portal. You can also dig around to find the right role IDs and assignment scopes, but there are lots of docs for that. If you have a lot of clusters it may be worth that effort. I'll do a follow up if anyone asks.

Once you find your AKS cluster, click Access Control (IAM) and add a role assignment. Search for your SP (the appId field will work, or the name as listed in the output of the `az ad sp` command earlier). Find an appropriate role. In my case, Azure Kubernetes Service Cluster Admin Role is what I was looking for. Select that, then make sure you Save/Add the role assignment.

![portal-role-assignment](img/aks-sp-portal-role-assignment.png "portal role assignment")

## Logging into the CLI with your newly minted SP

Now that our roles are assigned, let's jump back to the CLI. You can do this in your current CLI but, of course, this can be made a part of your deployment scripts through your deployment tool of choice - if it can run az-cli, it can be automated.

If you want to be absolutely sure it's working, clear all existing credentials first using `az account clear`. Once we've done that, let's get logged into the CLI with your SP. All of the required info for this came out of the output of the `az ad sp` command.

`az login --service-principal --username "your-service-principal-appId" --password "your-service-principal-password" --tenant "your-service-principal-tenant.whatever"`

Of course, if you used a certificate you'll need to make sure your cert is available to the cli and the command switches will be a bit different. Refer [here](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest){:target="_blank"}. You can ensure your login was correct by checking the output - under user.type, you'll see `servicePrincipal`. You can get back to this at any time with `az account list.`

## `kubectl` as an SP

Lastly, we need to get our aks credentials for kubectl:

`az aks get-credentials -g *your-resource-group-name* -n *your-aks-cluster-name*`

Make sure it all works with `kubectl get nodes`

![kubectl](img/aks-sp-cli-kubectl.png "cli")

:bowtie:

happy automating!
