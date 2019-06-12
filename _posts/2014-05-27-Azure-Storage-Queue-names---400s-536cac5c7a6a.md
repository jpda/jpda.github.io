---
title: Azure Storage Queue names + 400s
description: ''
date: '2014-05-27T15:24:21.000Z'
categories: []
keywords: []
slug: /azure-storage-queue-names-400s-536cac5c7a6a
---

Keep ’em lowercase. They’re DNS names, so while the _should_ be case-insensitive, they are, in fact not. So if you’re getting 400s creating queues (since Bad Request is so helpful, an all) — make sure all of your queue names are lowercase.

Here are all of the queue naming rules, from [http://msdn.microsoft.com/en-us/library/dd179349.aspx](http://msdn.microsoft.com/en-us/library/dd179349.aspx):

> Every queue within an account must have a unique name. The queue name must be a valid DNS name, and cannot be changed once created. Queue names must confirm to the following rules:

> A queue name must start with a letter or number, and can only contain letters, numbers, and the dash (-) character.

> The first and last letters in the queue name must be alphanumeric. The dash (-) character cannot be the first or last character. Consecutive dash characters are not permitted in the queue name.

> All letters in a queue name must be lowercase.

> A queue name must be from 3 through 63 characters long.