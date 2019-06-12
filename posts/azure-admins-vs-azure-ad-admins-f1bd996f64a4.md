---
title: Azure Admins vs. Azure AD Admins
description: ''
date: '2014-07-29T14:48:06.000Z'
categories: []
keywords: []
slug: /@jpda/azure-admins-vs-azure-ad-admins-f1bd996f64a4
---

This is a point that’s a bit ambiguous. I’m an Azure Service administrator, so I should be able to access the Azure AD associated with that tenant, right?

In a word, no.

#### TL, DR

If you need access to Azure Active Directory to add apps, users, etc — it’s pretty simple. It’s something we get a lot whenever one of our clients is using Azure, has granted us Azure service admin rights (E.g., Azure co-administrator), but we’re still asking for more access.

You can do this at the Azure portal ([http://manage.windowsazure.com](http://manage.windowsazure.com)) or, for adding a new user in your org or setting permissions to existing people in your org, the Office 365 Admin Portal ([http://portal.office.com](http://portal.office.com)).

Your Azure AD administrator (if you’re the creator of the account, this should be you, otherwise it’s time to track down the admin) just needs to add you. You can add

*   **A new account in your organization — **this creates a brand new account in your Azure AD tenant. No different from creating a new cloud-based user for O365.
*   **An existing Microsoft Account** — for sharing with the plebs who don’t have an Office account
*   **An existing organizational account in another directory **— for sharing with other organizations that use Azure AD (e.g., jpd.ms or cardinalsolutions.com).

Once the account is in Azure AD, you can set an access level. More info on access levels below.

#### Office Portal

Adding someone from the Office portal is easy. Open the portal as an admin, go to users and get crackin’ — but note, the Office portal only allows you to add users to your own organization. Could you imagine the chaos if you had options to add users from other orgs in here? LOL

![office portal](https://cdn-images-1.medium.com/max/800/0*e2FmgJHEwndpFunJ.png)

_But_ — you can add rights to an existing user from within the Office portal. Find the user in the list and you can set their access levels in there. Here’s my Live account, which was already added to my AD — I can set admin rights right from the portal.

![2-op](https://cdn-images-1.medium.com/max/800/0*nlOxY2GtkS041IUF.png)

#### Azure Portal

You can do everything from in here. Let’s dig in. Head into the Azure portal and find Active Directory in the left nav. If you don’t see your directory and you’re a member of multiple Azure subscriptions, click the Subscriptions filter to make sure you find the right one.

Once you’ve found your directory, click it and go to the Users header — here we’ll add a new user and grant them rights to AD. Alternatively, if you already see the account you want to grant access to, you can do that from in here as well.

![1-adduser](https://cdn-images-1.medium.com/max/800/0*Iq_u8PNfSSySNbuV.png)

Pick one of those three options (they’re outlined above) — if it’s not a valid account for the type you picked, when you try to go the next step, it’ll bomb out:

![badacct](https://cdn-images-1.medium.com/max/800/0*taUO61GUqD3D0u7J.png)

Otherwise, when you click next, you’ll be able to both set a name & display name for the new user (so that your Xbox name ‘N00b Slay3r 123’ doesn’t show up in your corporate apps) and grant an access level. Once you click the check, that new user is ready to go.

![2-user](https://cdn-images-1.medium.com/max/800/0*_pYRmAE8nSB7U4XT.png)

#### Role Play

Let’s talk about roles for a minute — you can find comparisons of each role here ([http://msdn.microsoft.com/library/azure/dn468213.aspx](http://msdn.microsoft.com/library/azure/dn468213.aspx)) — I’m not going to repeat any of that.

Global administrators — if you’re using Azure for anything beyond test/dev, or if you’re using it with Office 365/Intune/CRM, you probably want as few global administrators as possible. The **Global** part of global administrator is just that — quite global.

So what’s an admin to do when he/she has devs clamoring to add apps? I can’t seem to find anything ‘official’ about what access levels have access to add applications to your Azure AD — but it appears that User administrators do have this right. It makes sense, since ‘user administration’ is really principal administration, and all of these apps are new principals.

If you only want to give your devs access to add new apps, User Administrator might be a good role for them.