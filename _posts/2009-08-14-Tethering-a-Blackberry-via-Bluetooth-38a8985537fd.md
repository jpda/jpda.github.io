---
title: Tethering a Blackberry via Bluetooth
description: ''
date: '2009-08-14T21:19:41.000Z'
categories: []
keywords: []
slug: /tethering-a-blackberry-via-bluetooth-38a8985537fd
---

Last Friday, we made our summer pilgrimage to Michigan to visit family. This year we were heading up (farther) north to visit some extended family.

Let me digress; we left Friday morning around 7a, and arrived at her parents’ house around 830p. Midway through the day on Friday, I got an email from someone asking for a how-to document I had written, so I figured I’d polish up my draft and send it over the weekend.

Saturday morning, we headed five _additional_ hours north.

In such remote locations, internet connectivity is hard to come by — and so are other WiFi routers to leech from. So after some panicking, I decided to explore tethering my Blackberry to my laptop for the ultimate MacGyver’d internet connection.

As a disclaimer, I don’t know how different companies charge for tethering. This is my work phone, and since I needed a connection to do work, I figured any possible expense would be easily explained. I know that T-Mobile (my old mobile provider) charged a flat rate for everything, but who knows what AT&T does, especially with corporate accounts and such. Anyhow, you’ve been warned.

I installed Windows 7 RTM last Wednesday night before leaving, so I’m working with a pretty vanilla machine right now — Windows 7 RTM, Visual Studio\\SQL Mgmt Studio and some other development tools, Office 2010 and little else. Of primary importance for this little tutorial, I did NOT have the Blackberry Desktop software installed, and I still don’t know — it’s not a pre-requisite. I was able to get Bluetooth tethering working without any additional software — good if you’re unprepared and have no chance of an internet connection.

Anyhow, let’s get started. I’m using an AT&T Blackberry Curve 8310 (GPS) and a Lenovo T500 with a Broadcom Bluetooth chip.

Turn your phone’s Bluetooth radio on and pair it with your laptop. My PC installed a ‘Standard Modem over Bluetooth link’ driver once it paired.

Next, find your modem in Device Manager (Start à Run à devmgmt.msc à Modems à Standard Modem over Bluetooth link) or Phones and Modems in Control Panel.

![tether3](https://cdn-images-1.medium.com/max/800/0*27VdsTSnh1PYKz9j.png)

Open the properties of your modem and click the ‘Advanced’ tab. Type this in (including quotes) to the Extra Initialization Commands box: +cgdcont=1,”IP”,”wap.cingular”

![tether4](https://cdn-images-1.medium.com/max/800/0*a11oWsFc8iaVdPRO.png)

Now your modem is configured. Next we’ve got to create the actual connection. Head over to Devices and Printers.

![tether5](https://cdn-images-1.medium.com/max/800/0*RCSmDKUjhZjkUoL8.png)

You should see your Blackberry in there — right click and select ‘Create a dial-up connection…’

![tether6](https://cdn-images-1.medium.com/max/800/0*M8kqeBYtknIgCdIa.png)

Select your modem on the next screen, and use these settings:

Dial-up phone number: \*99#

Username: [WAP@CINGULARGPRS.COM](mailto:WAP@CINGULARGPRS.COM)

Password: CINGULAR1

Click next, then finish. Chances are it won’t connect. That’s ok, because there’s another setting we have to fix.

![tether7](https://cdn-images-1.medium.com/max/800/0*_bHWBQvD95P1fBpT.png)

Go to Network Connections (Start à Run à ncpa.cpl) and find your new dial-up connection. Right-click, open the Properties, and head to the Networking tab, then double-click Internet Protocol Version 4 (TCP/IP v4).

![tether8](https://cdn-images-1.medium.com/max/800/0*i5uIZFrnvmNrVJS1.png)

Click Advanced, and Clear the ‘Use IP header compression’ checkbox under PPP Link.

![tether9](https://cdn-images-1.medium.com/max/800/0*GEBzybIfD-hC2ZO5.png)

Now your connection should be ready to go. I’ve had good luck with both the password above and also with a blank username and password. Be sure to try a blank username\\password if the ones provided don’t work. Also, you may need to reboot for everything to properly take effect (particularly the modem installation — I had an error 734 and error 692 — rebooting fixed things right up).

Now, I’m on AT&T and my phone is an EDGE phone — prepare yourself for BLAZING 10 KB/s downstream (around 115kbps) and super-high latency — cellular networks are known for their brutal initial latency. Enjoy!