---
title: ‘No item exists at’ on public sharepoint site
description: ''
date: '2013-04-24T06:21:00.000Z'
categories: []
keywords: []
slug: /no-item-exists-at-on-public-sharepoint-site-3d7a7082819f
---

Here at WTFHQ, I’ve been working on building a SharePoint based public website for our company, dressed to the nines — master page, page layouts, themes, the works. I noticed, however, that I couldn’t edit or view properties of damn near _anything_. So I did what any SharePoint person does when confronted with a ​problem — I deleted the site collection.

Sufficiently proud with my ‘everything is a nail’ approach, my hubris soon turned to fury as I realized that the problem wasn’t rectified. Not content to say ‘I was wrong,’ I created an entirely _new_ instance of Office 365 to test my theory.

#### Surely, this is the problem.

Well, no. The problem kept on happening. Change master page, click ‘View Properties’ or ‘Edit Properties,’ and watch the world burn.

#### Seattle is your savior.

Fortunately, there is a quick fix (although it’s not something we should have to go through _anyway_). Head over into your Site Settings → Site Master Page (\_layouts/15/ChangeSiteMasterPage.aspx). Once you’re here, you want to set your System Master Page to seattle. It appears that _none_ of the other master pages will work — perhaps it’s because they’re HTML-based…I’d suspect it’s something to do with how the ID parameter gets handled, but since I’m exclusively SharePoint Online, access to ULS is _verboten._ You can try this bug out yourself — it’s happened on both developer preview sites & brand-new RTM E3 sites, so I’m pretty sure it’s not intended.

*   Go to the master page gallery or somewhere similar
*   Click arrow to get menu
*   Click ‘View Properties’ or ‘Edit Properties’
*   See/edit properties, no beef.
*   Go to edit system master page, pick something _besides_ seattle
*   Repeat. View/edit properties will blow up.
*   Go back and reset master page to Seattle.
*   Everything’s good again.
*   Lean back, put your feet up and say…’srsly, sharepoint…WTF?!’