---
title: Force NoDo on your Samsung Focus
description: ''
date: '2011-03-31T04:42:16.000Z'
categories: []
keywords: []
slug: /@jpda/force-nodo-on-your-samsung-focus-651a49dc66c0
---

This is more of an obligatory blog post, as I’m more interested in sharing my experience rather than the specifics of the hack.

I grabbed all of this from those wizards at xda-developers here: [http://forum.xda-developers.com/showthread.php?t=1012189](http://forum.xda-developers.com/showthread.php?t=1012189)

Basically, dev unlock your device (either via a paid developer account or through ChevronWP7).

Make a registry change to your Samsung Device, changing the Carrier ID string to 000–88. Rivera has released a packaged called SamsungRegistry.xap that you can sideload which changes that key specifically.

On your PC, download the USAIP.pbk file mentioned on the board. It’s a list of PPTP VPN connections you can use for free — they are demo accounts, so they disconnect after a few minutes.

The next step gets a little strange. Open the carrier portion of the settings pane, but don’t press anything yet — you’ll do that in a sec.

Connect to one of the PBK connections. I connected to the one suggested, EuroIP PPTP Hungary.

Plug your phone in via USB.

Open the Zune software, and navigate over to the manual update section (Phone –> Sync Settings –> Update).

This is where it gets odd — on your phone, you should still have the carrier settings screen open. After a few seconds, turn your data connection off. If you timed it properly, it will tell you an update is available for your phone. Otherwise, it will tell you there were some problems connecting to the update server. Do this _ad nauseum_ until you get it right.

There are a couple of things that I did that may make your experience better; I’ll detail those in my next post.