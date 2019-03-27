---
title: "Force NoDo on your Samsung Focus redux"
date: 2011-03-31T05:00:58.000Z
author: "John Patrick Dandison"

---

Per the norm, ATT is bringing up the rear in consumer appreciation, leaving us waiting indefinitely for the March WP7 update. I got my wife an HD7 for Valentine’s Day and T-Mobile is already sending out the update to those…

So I got a little impatient. There are lots of tutorials on how to force an update, so I’ll hit the high points. Basically — it’s a pain in the ass.

I have two pieces of advice:

1: If you are planning to become a registered developer, DON’T use the ChevronWP7 unlocker. ChevronWP7 requires that you install a certificate (ChevronWP7.cer, usually), via email or through a website.

Unfortunately, when you developer uinlock a phone, it also wants to install a certificate — and it appears that if your phone already has developerservices.windowsphone.com certificate (the bogus one installed for Chevron), it skips right over the actual dev unlocking process and tells you it’s ok…a quick check on the website, however, shows your device truly isn’t a registered development device…so, what to do, you ask?

The only way to get rid of certificates (that I have found) is to wipe the phone…so yeah, it’s a bit of a pain. Luckily, wiping the phone does not wipe the updates — those seem to be fully merged into the firmware, so there is no differencing going on. After I wiped

2: Break out a stopwatch. I used the Hungarian PPTP VPN Proxy method outlined by the wizards at the xda-developers forum (outlined in shoddy detail [here](http://jpd.ms/post/2011/03/30/Force-NoDo-on-your-Samsung-Focus.aspx)).

The first 15 times I tried this, I kept on getting the ‘update server could not be reached’ timeout message. It was incredibly annoying.

Then I got the February 2011 ‘pre-update’ update. Filled with renewed spirits, I trudged forward, convinced that the ‘next’ time I tried this I’d get the update.

Fast forward another fifteen or so iterations, and I started losing faith. One forum post suggested timing it by seeing how long it takes to timeout, then subtracting a second or so from that — and that’s the time you turn off the data connection…here’s my experience with that:

The first time I tried that, I followed the steps (turned off data connection while the Zune software was looking for updates)…that took a full 64 seconds before timing out. The next iteration, however, I kept the phone’s data connection on, and it timed out after only about 32 seconds. The next time through, I turned off my data connection on the phone right at 30 seconds — and it worked. But don’t ask me to do it again.
