---
title: 'OCS 2007 R2, Empathy & Ubuntu 10.04'
description: ''
date: '2010-08-26T04:47:51.000Z'
categories: []
keywords: []
slug: /@jpda/ocs-2007-r2-empathy-ubuntu-10-04-d38cf811a01b
---

I’ve been using Ubuntu Lucid (10.04) on and off for a while, mainly exploring some of the MonoDevelop IDE. Since my company uses Office Communicator for IM, it’s important to be able to connect up to our OCS at the office.

The pidgin-sipe plugin works great with pidgin, but pidgin doesn’t integrate well with the MeMenu\\indicator-applet new to 10.04. Since Empathy and Pidgin both understand a good portion of libpurple, the consensus around the internets was that simply installing the pidgin-sipe package (sipe only, not the entire pidgin app) would get Empathy connected.

sudo apt-get install pidgin-sipe

I got the packages and all was good, Empathy saw the plugin (after a log off\\log on), and I was away. I put in as much information as I could — username, server name, protocol, and all of my OWA information.

Nothing.

Tweaked a couple settings.

Still nothing.

I decided to take a different look at my username. I remembered pidgin used a different syntax for username, and I suspected that may have something to do with it here — especially since my SIP name is on a different domain from my username.

Rather than burn a bunch of cycles, I used this syntax for username, and started getting more meaningful results:

john.dandison@sipDomain.com,logonDomain\\john.dandison

Decided to try grabbing our CA, since our internal OCS pool is self-signed. Copying the root CA to /etc/ssl/cert did the trick — now I was connected. There was absolutely no error message or dialog regarding the cert, so it was kind of a shot in the dark — but it worked.

Still, that was only when VPN’d into the network. Since most OCS installations have an internal address and an external address, I didn’t want to have to switch server addresses each time I was on or off the network. I dropped the VPN and plugged in the public server name, but as soon as it would try to connect, it would immediately kick back out. The debug logs showed that the server was dropping the connection almost immediately.

Oddly enough, I had totally forgotten that we have SRV DNS records (as I’m sure most large companies do) — as soon as I took the server name out, I was auto-connected inside or outside of the network. I moved to a different ubuntu box (without the root CA installed), and I left the server name field empty. I got the ‘Connecting…’ message, which would eventually time-out, so I installed the CA cert — voila, all was working, without any manual configuration.