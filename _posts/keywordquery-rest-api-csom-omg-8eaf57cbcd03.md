---
title: 'KeywordQuery, REST API & CSOM, OMG'
description: ''
date: '2013-12-05T00:00:00.000Z'
categories: []
keywords: []
slug: /@jpda/keywordquery-rest-api-csom-omg-8eaf57cbcd03
---

A weird issue passed across my desk today — people searches done via REST work fine, but done through CSOM return no results. It sounds permissions based almost immediately, but I decided to look.

And it is permissions (of course).

#### Searching for Users

Users   
 _who haven’t been to your site before_ really only live in the User Profile Service. It appears as though apps really don’t have permission to search that, even though it’s usually available to all authenticated users. Oddly enough, if you go in   
 _without_ a context, you’ll get results as well. Moving along, then…

#### Tenant Full Control? Good luck.

Asking for tenant full control rights is one way to make this work. In fact, if you want the app to do searches with its own token (without a user context), this \*may\* be the only way. Of course, chances are your organization/client probably won’t want to give you that kind of access. Fortunately, there’s a better way. In the AppManifest, there’s a permission request under search:

<AppPermissionRequest Scope="http://sharepoint/search" Right="QueryAsUserIgnoreAppPrincipal" />

And that’s it — now your searches are done as the user, thus avoiding the entire issue. Not that this doesn’t bring up its own set of problems (app-only access, etc), but it’s an option.