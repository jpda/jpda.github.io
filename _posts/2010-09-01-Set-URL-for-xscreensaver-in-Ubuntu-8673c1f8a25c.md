---
title: Set URL for xscreensaver in Ubuntu
description: ''
date: '2010-09-01T03:48:49.000Z'
categories: []
keywords: []
slug: /set-url-for-xscreensaver-in-ubuntu-8673c1f8a25c
---

The phosphor screensaver, part of the xscreensaver-data package, is a screensaver that emulates an old terminal — it’s pretty sweet, and I like to have the feed from the National Hurricane Center running during the summer.

Unfortunately, with gnome-screensaver, it’s impossible to change those settings. So there are two things you can do. Here’s how I did it, there’s probably a better way.

Look for a hidden file in your home folder (Ctrl-H) called .xscreensaver. If there is one, then open it and skip down to the bottom. If there isn’t one, you’ve got some work to do.

Initially I wanted to just replace gnome-screensaver. Ultimately, I ended up reinstalling it — and that’s how I got this to work. In short, you need to generate the .xscreensaver file in your home folder.

Here’s what I did (instructions [here](http://www.jwz.org/xscreensaver/man1.html "Xscreensaver man pages") under ‘Using GNOME’)

Remove gnome-screensaver (sudo apt-get remove gnome-screensaver)

Install xscreensaver (sudo apt-get install xscreensaver)

Edit /usr/share/applications/gnome-screensaver-preferences.desktop and change the _Exec=_ line to say Exec=xscreensaver-demo

Fire up the screensaver preferences pane. Make some changes. You can close out of that. Check for a .xscreensaver file in your home folder.

**So you’ve got your .xscreensaver file — let’s make some changes.**

It’s pretty simple here — find this block:

textMode: url (url, file or program)  
 textLiteral: XScreenSaver  
 textFile: (any text file)  
 textProgram: fortune (could be anything here)  
 textURL: [http://www.nhc.noaa.gov/mobile/MIATWDAT.html](http://www.nhc.noaa.gov/mobile/MIATWDAT.html) (text-only Atlantic Tropical Discussion from the NHC)

Set your textMode property to url — and set the textURL property to the URL of your text. I’ve found that it seems like it may have a simple RSS parser in it, as pointing it to an RSS feed seems to work pretty well.