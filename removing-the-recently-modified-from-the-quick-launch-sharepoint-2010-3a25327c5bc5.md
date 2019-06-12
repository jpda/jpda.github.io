---
title: Removing the ‘Recently Modified’ from the Quick Launch — SharePoint 2010
description: ''
date: '2010-01-26T02:25:00.000Z'
categories: []
keywords: []
slug: >-
  /@jpda/removing-the-recently-modified-from-the-quick-launch-sharepoint-2010-3a25327c5bc5
---

Update (7/29): Now that SP2010 has gone RTM, editing files directly on the filesystem is not a best practice — in fact, it runs a lot of risks — at best, you’ll lose your changes after an update or a service pack, at worst you’ll screw up your entire installation.

There are two good options:

If it’s only on a page or two, drop a Content Editor Web Part on the page and plug in this CSS (from RosalynA):

<style>

.s4-recentchanges

{

DISPLAY:none;

}

</style>

The other option would be to make the change with SharePoint Designer — this breaks the page from site definition, leaving the filesystem alone, and saves the changes in the database. If you ever need to go back to stock, you simply ‘reset to site definition’ and the page goes back to normal. This protects you from changes which occur during service packs or updates (thanks Scott).

**UPDATE**: I saw where this post got posted on the MSDN Social — so, welcome MSDN Users. Also, it was pointed out that directly modifying server files is dangerous, primarily because you don’t get versioning\\ghosting, nor is the page\\file backed up. BE ABSOLUTELY SURE that anything you update in the 14 folder (or the 12 folder, for that matter), is properly backed up — and please, please, use a UAT environment — don’t be ‘that guy’ that tests stuff in production. Anyway, proceed.

I’m getting married in June — my fiance has been using [my SharePoint 2010 environment](http://wedding.johndandison.com) to build a site providing our guests with the information they need if they’re planning to make the trip.

Anyway, a basic team site uses the Wiki Page template for new pages, so we had a problem — this ‘Recently Modified’ web part that would appear above the quick launch on wiki pages (I stole this image from [wssdemo.com](http://wssdemo.com), since I already took it off of mine).

![image](https://cdn-images-1.medium.com/max/800/0*IA9oObSPoBeOT98b.png)

After some searching, no one seemed to have had this issue before, so I went digging around to see what I could find. After looking through the master pages, content types, and page layouts, I decided to head into the 14 folder (program files\\common files\\microsoft shared\\web server extensions\\14) in search of relief.

After a bit, I came across the ‘DocumentTemplates’ folder under 14\\template — you’ll find a wkpstd.aspx file in there. The page inherits from ‘Microsoft.SharePoint.WebPartPages.WikiEditPage,’ so that made me suspect I was onto something.

Anyway, open that up and find the ‘SharePoint:RecentChangesMenu’ control. You could probably delete it, but being a server control, I figured just slapping a visible=”false” on the end of the tag should work. Added visible=”false”, saved the file, and refreshed my page — sure enough, no more ‘Recently Modified.’