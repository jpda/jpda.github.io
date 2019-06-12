---
title: Power Management in Windows 7/8
description: ''
date: '2013-04-11T05:53:00.000Z'
categories: []
keywords: []
slug: /power-management-in-windows-7-8-a049369e0c34
---

Recently, my laptop has been frustrating me, quite considerably. If I stopped using it for a couple of minutes, it would go to sleep. Monitors off, network connection dead, completely off. I set my sleep timers to never. Same thing — two minutes in, completely asleep.

#### wat, indeed.

It was somewhat inconsistent. Most of the time (like 80%), it would sleep after two minutes, regardless of settings…but ever so often, it  
_wouldn’t sleep_ until the prescribed time. What’s worse than a problem? An inconsistent one. Fortunately, there’s a solution. It’s easy (and dumb), but it works.

#### A note on ‘power management.’

So there’s this ‘thing’ in PCs called power management. If you’re reading this blog, chances are high you know what it is, so I’ll just skip any explanation, but it is effectively what is sounds like — management of power. And technically, it  
_does_ manage power — as in, it turns things off and on, or changes power states. Just not with any rhyme or reason.

#### Laptop users with docks — you’ll experience this before anyone else.

I found this issue only to affect my laptop in its dock — my home desktop never experienced this issue, nor did any  
_non-docked_ laptops. That’s a key here — external peripherals. It seems there’s a power setting — ‘sleep after x minutes  
_unattended_.’ It’s not typically exposed through Power Options, but it’s the key to this problem. It appears that waking your laptop through an external keyboard or mouse (as I was doing) isn’t considered an ‘attended wake’ by Windows — so it would sleep it after a couple of minutes.

#### Easy Workaround & Complete Fix

The easiest, least-invasive way to fix this is to just wake up your laptop with the power button, either on the laptop or the dock (at least, the dock button worked for me, HP Elitebook 8760w). That’s great, if you remember — your normal sleep timers will all work fine and you won’t feel like sending your laptop on a magic carpet ride through the nearest window. The permanent fix isn’t much worse, but it does involve a new registry key.

#### Fix it for good.

Thanks to lots of googling, I found that the marvels over at [sevenforums](http://www.sevenforums.com/tutorials/246364-power-options-add-system-unattended-sleep-timeout.html) had published the registry settings necessary to get the mysterious ‘system unattended sleep timeout’ to appear in power options.

Pretty simple — head over to your registry (srsly, if you break something, I don’t want to hear about it) and dig down into:

`[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\238C9FA8–0AAD-41ED-83F4–97BE242C8F20\7bc4a2f9-d8fc-4469-b07b-33eb785aaca0]`

You can make sure you’re in the right place if you check the FriendlyName key — it’ll say something like ‘System unattended sleep timeout’ (or whatever it is in your language). Change the Attributes DWORD to 2, reopen Power Options (you may need to reboot) and you should see the option now available. It’ll probably something _completely silly_ like two minutes.

#### See? Silly.
