---
title: Using Organizational Accounts for Azure Subscription Administration
description: ''
date: '2015-03-22T04:06:36.000Z'
categories: []
keywords: []
slug: >-
  /using-organizational-accounts-for-azure-subscription-administration-8df027cd146c
---

Here’s one we get frequently — no one wants their enterprise Azure account administered by someone’s Xbox Gamertag. ‘noobslayer@hotmail.com’ doesn’t look great during a review of admins, nor is it easy to immediately know who the slayer of noobs may be. Organizational accounts are so much better for management and control over who’s handling your subscription.

There are two main scenarios — adding a user from a directory that’s already connected to the subscription, and also adding a user from _a different_ directory (think managed services — managing a client’s Azure subscription using your existing organizational account).

To make it easier, let’s start with some definitions.

#### Definitions

* Organizational account — also known as an Azure AD account. Ends in your own domain (like jpd.ms or cardinalsolutions.com) or the out-of-the-box managed domain (yourorg.onmicrosoft.com). **If you’re using Office 365 today, that’s an Azure AD/Organizational Account.**
* MSA — Microsoft Account, like someone@hotmail.com, @outlook.com, @live.com, etc.
* Tenant — organization, specific instance of Azure AD for your organization

And the two scenarios for today:

* Administering your subscription with an account from your organization
* Administering another subscription with an account from your organization

#### Subscriptions + Azure AD Tenants

There is a bit of confusion surrounding how these two seemingly unrelated products work together. You’ll get an Azure AD tenant as part of your setup process of a new Azure Account. That tenant’s domain will end in .onmicrosoft.com — of course you can add your own domains (and if you plan SSO, it’ll be a requirement), but out of the box, with zero configuration, you’ll have a .onmicrosoft.com Azure AD tenant. That tenant’s only administrator should be the MSA you used to create your Azure subscription. This is an important distinction, because if your tenant and sub were provisioned differently, you may need to make sure your MSA is an administrator of your Azure AD tenant.

#### Administering your own Subscription

#### Linking your Azure Subscription to your Azure AD Tenant

This one is quick and easy. Provided you’ve setup your Azure AD domain (there are plenty of tutorials for doing this), it’s a two step process. First we need to link your directory with an Azure AD organization/tenant. Start at [https://manage.windowsazure.com](https://manage.windowsazure.com) and head down to settings. If you’re using the same MSA that you created your Azure subscription with, your MSA should also be an administrator of your Azure AD tenant.

![Under settings → subscriptions, you’ll find a list of your subscriptions.Under settings --> subscriptions, you'll find a list of your subscriptions.](/img/0_fEIdli12xGBjxumY.png)
Under settings → subscriptions, you’ll find a list of your subscriptions.

Now, find your subscription in the list, and at the bottom you’ll see ‘Edit Directory’ in the dark bar.

![Find this to link your subscription to a directory.Find this to link your subscription to a directory.](/img/0_QVni10Fh5sdvzwEv.png)
Find this to link your subscription to a directory.

This will bring up a new box, listing all of the **Azure AD Tenants** (not Azure subscriptions!) your MSA has access to. In your case, you may only have one. If you have more than one, pick the one that looks familiar or that contains the organizational user you want to have access to that subscription (e.g., if I want joe@company.com to have access to my sub, I need to find the Azure AD tenant that has joe@company.com).

![You’ll see a list of all of the Azure AD Tenants that MSA has access to.You'll see a list of all of the Azure AD Tenants that MSA has access to.](/img/0__4TTBmMYeQus-BPW.png)
You’ll see a list of all of the Azure AD Tenants that MSA has access to.

Once you’ve picked yours, you’ll need to confirm the change. No sweat.

Next we need to add the administrators.

#### Adding an Organizational Administrator

If your Azure subscription got linked up to the proper directory in the last step, this is just as easy as adding a new administrator. Under Settings → Administrators, click Add at the bottom. You should see a simple form to search for a user:

![Settings → AdministratorsSettings --> Administrators](/img/0_laZmNtLQKqMam-Po.png)
Settings → Administrators![You’ll notice MSAs resolve as MSAs…You'll notice MSAs resolve as MSAs.](/img/0_MeJUkKLA-Kfly4-3.png)
You’ll notice MSAs resolve as MSAs…![…and organizational accounts will resolve as org accounts....and that organizational accounts will resolve as org accounts.](/img/0_bRUxK_q6N92TxZNu.png)
…and organizational accounts will resolve as org accounts.

Once you’ve typed in the proper account that needs access, check the boxes next to the subscription(s) you want that account to be able to co-adminster. If the name resolves, you’re finished.

#### Using Org Accounts to Manage _Another_ Org’s Subscription

Take this scenario — you are a managed service provider offering Azure resources and subscriptions as part of your management package. This implies you need to sign into other Azure subscriptions and manage them, as a co-administrator. But your organizational account is already using Azure AD, so you (and any of your employees) still need to use organizational accounts to administer customer subscriptions (instead of MSAs). Fortunately, it’s possible, but with a few more hoops.

Here are some dummy account names to make this more concrete:

a) Your business is Super Azure Consultancy, or SAC, and your domain is @sac.local (this deliberately won’t resolve and won’t be a domain that you’ll find in any Azure AD tenant. The domains need to be internet-resolvable).

b) You offer managed Azure subscriptions to your clients. Client A is Larsen’s Biscuits, @larsen.local

c) Larsen’s has an Azure subscription, tied to their Azure AD tenant (per the instructions in the section above).

d) You need to have your employee, mark@sac.local, administer Larsen’s Azure subscription using his @sac.local account.

#### Getting another Org’s Users into Your Org’s Tenant

#### Add a user from Tenant A to Tenant B

First, we need to add mark@sac.local to the larsen.local Azure AD tenant. This sounds simple on the surface, but is a bit trickier. In short, you need to sign into the Azure management portal with an account **that has access to both the source directory (sac.local) and the target directory (larsen.local). This is likely easiest using an MSA.**

Either create or use a MSA and add that MSA as a global administrator to each  **_Azure AD Tenant, not just the Azure subscription_** _(_see [here](http://jpd.ms/azure-admins-vs-azure-ad-admins/ "Azure Admins vs. Azure AD Admins") for how to do that).

Next we need to add Mark (mark@sac.local) to the Larsen tenant. Head into Azure AD in the Management Portal, click Users, then Add:

![Management Portal → Azure AD → Tenant → Users → AddManagement Portal --> Azure AD --> Tenant --> Users --> Add](/img/0_UjyRKUaNEx9ORpvp.png)
Management Portal → Azure AD → Tenant → Users → Add

You’re going to want to pick ‘**User in another Windows Azure AD directory**.’ Next we’ll type in mark@sac.local — if it works, you’ve done it successfully. You can add the user as a user, unless that user is going to be administering Azure AD as well.

If you get a message that the current user doesn’t have access to the directory, you’ll need to **be sure the MSA has admin rights to both Azure AD tenants**, using the info in the link a little higher.

![You’ll see this message if you’re not using an account that’s an administrator in BOTH Azure AD tenants.You'll see this message if you're not using an account that's an administrator in BOTH Azure AD tenants.](/img/0_Keq5BGFNcnHCGlJW.png)
You’ll see this message if you’re not using an account that’s an administrator in BOTH Azure AD tenants.

#### Add External Tenant user as Co-Administrator

At this point, you should be able to follow the directions from "Adding an Organizational Administrator" above, but be sure to use the new user you just added (e.g., mark@sac.local). This will allow mark@sac.local to administer the Azure subscription linked to the @larsens.local Azure AD tenant.

Confused yet?

Feel free to reach out with any problems.
