---
title: Hi Slade — are you referring to the appRole in the manifest?
description: 'appRoles: ['
date: '2019-03-01T03:20:03.359Z'
categories: []
keywords: []
slug: /@jpda/hi-slade-are-you-referring-to-the-approle-in-the-manifest-32f66d21083e
---

Hi Slade — are you referring to the appRole in the manifest? In that case it’s the ID of the appRole that we created, which we assign our MSI principal into. It shows up in the appRoles array:

appRoles: \[

{ id: “appRoleId — a guid”, description: “do all the things” …<snip>… }