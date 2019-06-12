---
title: IE9 Web App Jump Lists
description: ''
date: '2010-09-16T04:32:00.000Z'
categories: []
keywords: []
slug: /ie9-web-app-jump-lists-b4455b9824d3
---

The first IE9 beta was released today! That said, [go grab it](http://www.microsoft.com/ie) and try out some of the new stuff.

One thing I noticed right off the bat was how tabs could be torn off of the main window and pinned to the taskbar, which is sweet in its own right — but there’s more. Thanks to the peeps at Channel 9 — they’ve got a great demo of the jump list and other capabilities of ‘web app’ mode in IE9.

One of the sweetest parts of the Windows 7 taskbar is the Jump List. Jump Lists let you get to what you want in an application fast — and now they’ve extended that to the web. Not only can you specify your own jump lists, you can customize the icon as well as the colors of the navigation buttons, a major part of the new IE9 UI.

So how do we do this? Simple. Drop this code into the tag of your site, customize the links, and you’re off. Everything’s in here — make as many links as you’d like. You can set the window’s initial size (msapplicaiton-window), the navigation button colors and icons (msapplication-navbutton-color), all kinds of stuff. I’m still getting into it, so as I discover more, I’ll post here.

```html
<meta name="application-name" content="johndandison.com" />
<meta name="msapplication-tooltip" content="c#, sharepoint and other pointy things" />
<meta name="msapplication-window" content="width=1024;height=768" />
<meta name="msapplication-task" content="name=Latest Posts;action-uri=./;icon-uri=/themes/illacrimo/favicon.ico" />
<meta name="msapplication-task" content="name=Archived Posts;action-uri=./archive.aspx;icon-uri=/themes/illacrimo/favicon.ico" />
<meta name="msapplication-task" content="name=Contact;action-uri=./contact.aspx;icon-uri=/themes/illacrimo/favicon.ico" />
<meta name="msapplication-navbutton-color" content="#003466" />
<meta name="msapplication-starturl" content="./" />
```

![image](/img/0_QlnKMmNQNwtU2O1w.png)
![image](/img/0_m9XjyLHdPM32bQ7H.png)
