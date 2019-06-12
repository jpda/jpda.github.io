---
title: Init.js Library Not Registered — Office 2010 x64 & SharePoint 2007
description: ''
date: '2010-04-16T17:07:00.000Z'
categories: []
keywords: []
slug: >-
  /init-js-library-not-registered-office-2010-x64-sharepoint-2007-7024c13c6483
---

So I’ve been running Office 2010 x64 for quite some time now — through alpha, beta and the RC. Now, I’m waiting for RC with the hopes that this has been fixed.

Seems there’s a little something wrong with the Name ActiveX Control that SharePoint uses for OCS presence info. Whenever I run any SharePoint site, I get stuck with a bunch of JavaScript errors, which in turn break other scripts, leading to a less-than-optimal experience.

Does this look familiar?

![image](https://cdn-images-1.medium.com/max/800/0*nbGrPeXbQNMmZMaT.png)

Debugging the page gives a bit more insight, but not much:

![image](https://cdn-images-1.medium.com/max/800/0*52q5_9DVwPPsWPdj.png)

So we know it’s the IM Presence Control — (on a side note, this is why internet-facing SharePoint sites you may come ask you to run ‘Name ActiveX Control’).

I did some fishing through the IE Add-ons, only to realize that the NameCtrl is missing — then it hit me…

### The Problem

I’m running Windows 7 x64, and the x64 Office 2010 clients. Where does this lead us? The x64 version of Internet Explorer.

Hit your start menu and type ‘internet explorer 64’ — you should see it pop up in the menu.

![image](https://cdn-images-1.medium.com/max/800/0*3uLGN4IkO4R7-ViR.png)

Fire this up, and browse to your SharePoint site — you’ll notice an error-free interface — no JS errors — meaning any of your additional scripts (like the one we use for navigation) will work properly.

So take a look into the Add-Ons in IE x64 — you’ll see the NameCtrl ActiveX control listed in there.

Which makes sense, if you think about it — the default IE in Windows x64 is 32-bit IE, ironically to maintain plugin compatibility…64-bit plug-ins aren’t going to work in 32-bit IE, so it all falls into place.

But the irony here leaves us with a new problem — do we use IE x64 for everyday use? Or find a 32-bit NameCtrl to install into 32-bit IE?

Of course, there’s one other problem — no presence. Even though the NameCtrl gets loaded and is available to the DOM, it doesn’t seem to be communicating with Office Communicator very well (I’m running 2007 R2 — x86, since there isn’t an x64 version)…none of my contacts have proper presence.

### The Fix

Luckily, there’s an easy fix. Fire up an administrative command prompt. Navigate over to your 32-bit Program Files folder, and find Office in there — C:\\Program Files (x86)\\Microsoft Office\\Office14.

In here, you’ll find Name.dll and a few other files. At the command line, type this:

regsvr32 name.dll

Hit enter — you should get a notification that DllRegisterServer succeeded:

![image](https://cdn-images-1.medium.com/max/800/0*0OSY5Eu7eUz5u4UN.png)

Kill all of your Internet Explorer windows, and open the 32-bit version (the default) back up. Open up the IE Add-Ons, the previously-absent NameCtrl ActiveX control should now be listed…open your SharePoint site and enjoy.

![image](https://cdn-images-1.medium.com/max/800/0*Q-Xt-kCEORGeuvSb.png)