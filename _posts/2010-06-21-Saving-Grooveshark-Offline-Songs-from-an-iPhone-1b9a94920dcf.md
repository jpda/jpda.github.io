---
title: Saving Grooveshark Offline Songs from an iPhone
description: ''
date: '2010-06-21T12:58:50.000Z'
categories: []
keywords: []
slug: /saving-grooveshark-offline-songs-from-an-iphone-1b9a94920dcf
---

I probably shouldn’t write this, as I love the Grooveshark service and happily paid my $36 for the year. Got the desktop client, the iPhone client & the BlackBerry client. Today, I started wondering how and where the offline songs were stored — mainly because I’m on a slow connection and would love to get these tracks on my PC, where I can add them to my iPod for true, offline listening.

So I did some looking around via SSH (you need a jailbroken iPhone for Grooveshark, so I assume you’re capable of getting SSH installed and running if you’ve got Grooveshark).

This is all you need:

A pc\\mac\\laptop\\anything with an SSH client

WinSCP (for Windows), probably the best free FTP\\SCP client around

Fire up the Grooveshark app, and start playing one of your Offline songs (this probably works for online songs as well, I just haven’t tested it yet).

Open WinSCP and navigate to /tmp (root directory tmp, not the root user tmp) — you’ll find an mp3 in there called temp.mp3. Copy this down to your machine and give it a recognizable name — and that’s it. Open the MP3 and you’ll find all the ID3 tags filled out (and album art too, if it has any) with the Grooveshark info.

You have to actually start playing the song you want to download, as that’s what generates the temp.mp3 file — I’m still digging looking for the master database, but I’d suspect it’s encoded and stored somewhere, probably a BLOB in a database engine somewhere.

Lastly — don’t steal music. Paying ridiculous amounts of money for music is silly, but please support Grooveshark by going VIP, and be sure to support your artists.

![image](/img/0_aeD-pXo1Ncj4Ze7c.png)
![image](/img/0_S59RlDhYJV-6__xf.png)
