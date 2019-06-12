---
title: TenantDbContext for Table Storage
description: ''
date: '2014-12-11T21:45:24.000Z'
categories: []
keywords: []
slug: /tenantdbcontext-for-table-storage-3e35a5f8e55e
---

For anyone who’s used the ASP.net MVC templates with multi-organizational authentication, you’ll inevitably end up with a bunch of generated entity framework goo for keeping track of organizational certificate thumbprints for orgs who have logged into your app. This is lame. We’re creating two tables with a single column each in _SQL?!_ I’ve never heard of a better use of table storage. Not to mention I’ve got to now pay for a SQL Azure instance, even if my app doesn’t need it.

This speaks to a larger issue — how frequently are we, as developers, using SQL by default? Do we really _need_ relational data? Are we enforcing constraints in our service layer as we should? We are?! This makes SQL even _more ridiculous_ in this scenario.

I decided to build one that uses table storage. You’ll need a few things

* the source
* update your web.config to indicate the issuer registry type

The VS solution is on github here: [https://github.com/jpda/net-table-issuer-registry](https://github.com/jpda/net-table-issuer-registry)

It’s dependent upon Azure configuration and Azure storage. Licensed under MIT, if you find it useful I’d just ask you drop me a line and let me know what neat thing you’re working on!
