---
title: Modifying Screensavers in Ubuntu 10.04
description: ''
date: '2010-09-01T03:20:00.000Z'
categories: []
keywords: []
slug: /modifying-screensavers-in-ubuntu-10-04-cc82f2161347
---

This may not be the best way to do this, but it’s the way I found. Suggestions are always welcome.

I was trying to configure a particular screensaver in Ubuntu Lucid 10.04. For whatever reason, the GNOME developers, in their ‘infinite’ wisdom, decided to can xscreensaver in favor of gnome-screensaver — and they left out any way to configure a particular screensaver (for instance, choosing a folder besides your ~/Pictures folder for a slideshow).

After combing the web for a while, I found that gnome-screensaver supports most of the xscreensaver packages, so I installed these:

xscreensaver-data  
 xscreensaver-data-extra  
 xscreensaver-gl  
 xscreensaver-gl-extra  
 $> sudo apt-get install xscreensaver-data xscreensaver-data-extra xscreensaver-gl xscreensaver-gl-extra

Then I read up on editing these files by hand. From what I can tell, you’ll find a lot of .desktop files in the /usr/share/applications/screensavers folder.

Each of these .desktop files is a ‘hack,’ or ‘screensaver theme.’ This is where each screensaver is configured — namely, each _configuration_ of a screensaver is stored.

So the easiest thing to do is either to edit this directly, or copy the original and start working on the copy.

I copied the Phosphor saver, since I like it and it can be pointed to a web server or any other source of text. The default is some Ubuntu feed, which is boring and lame.

So I opened the copy and got to work. Here it is in it’s original form:

\[Desktop Entry\]  
 Name=Phosphor  
 Exec=/usr/lib/xscreensaver/phosphor -root  
 TryExec=/usr/lib/xscreensaver/phosphor  
 Comment=Draws a simulation of an old terminal, with large pixels and long-sustain phosphor. On X11 systems, This program is also a fully-functional VT100 emulator! Written by Jamie Zawinski.  
 StartupNotify=false  
 Terminal=false  
 Type=Application  
 Categories=Screensaver  
 OnlyShowIn=GNOME;

Pretty simple, eh? Make your changes in here — I wanted to change the scale and the font of the text.

So I made my changes — now it looked like this:

\[Desktop Entry\]  
 Name=Phosphor Droid NHC  
 Exec=/usr/lib/xscreensaver/phosphor -root -scale 2 -font ‘Droid Sans’  
 TryExec=/usr/lib/xscreensaver/phosphor  
 Comment=Draws a simulation of an old terminal, with large pixels and long-sustain phosphor. On X11 systems, This program is also a fully-functional VT100 emulator! Written by Jamie Zawinski.  
 StartupNotify=false  
 Terminal=false  
 Type=Application  
 Categories=Screensaver  
 OnlyShowIn=GNOME;

But no matter what I did — including rebooting the machine and running dpkg-reconfigure did any good.

I did some digging, and found the desktop.en\_US.utf8.cache file under /usr/share/applications

This file seems to be full of any config file that uses the .desktop nomenclature — all the GNOME desktop related stuff — like screensavers.

I moved this out to the desktop and fired up the screensaver applet. Sure enough, my new entry was there. Not only that, if you edit the .desktop files, you can check your changes without having the reload the screensaver applet — just click another one in the list, then back to your new one to check the settings.

I assume that running without the cache file will probably cause a bit of fetching when using GNOME apps, so it’s probably a good idea to make sure the info from your .desktop files ends up in your cache file. If I can find a different way to regenerate the cache file, I’ll be sure to update here.