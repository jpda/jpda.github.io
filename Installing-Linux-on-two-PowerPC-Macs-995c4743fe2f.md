---
title: "Installing Linux on two PowerPC Macs"
date: 2009-12-18T21:50:00.000Z
author: "John Patrick Dandison"

---

As anyone who regularly reads, I’m a Microsoft kinda guy — I use Windows, Office, SQL Server, IIS, Visual Studio and SharePoint. Staying within a single ecosystem makes it easy — things work together, are made for each other and it creates a very seamless experience. But that doesn’t mean I discriminate.

For example, I have an iPod. It’s actually my third one. I have an iPhone too. And a BlackBerry (granted, this doesn’t count because I HAVE to use this, and there are no other options at my company). I’ve got a few PCs here at home — two Lenovo laptops, a Motion tablet, a Dell XPS 420 and an old Vaio. I use Windows 7 on everything (except the Motion tablet, which uses Vista) or Windows Server for the DC and Hyper-V box. But in addition to the stable of machines here at the house there are two unassuming, yet ominous contributions: a Powerbook G4 12”, and a Power Mac G5.

The PowerBook is somewhat of a dog — it’s a 12”, all aluminum notebook. Aluminum is nice and ‘industrial looking,’ until it gets hot and you can feel it through your clothes (god forbid you actually USE this on your lap — sterilization city).

The Power Mac is a step up — dual PowerPC 970s at 1.8Ghz, 2GB DDR, Radeon 9600 and a 320GB sata disk. For a machine that’s five years old, those specs are pretty nice. The Power Mac G5 moniker is somewhat misleading — the PowerPC 970 is the little brother to the Power4 chip, not the Power5.




![powerpc-970](http://jpd.ms/wp-content/uploads/migrated/powerpc-970_thumb.jpg)





![Power4 _mod2fn](http://jpd.ms/wp-content/uploads/migrated/Power4_thumb.jpg)



The biggest problem with these machines is simple — they both use IBM’s PowerPC architecture. The last version of Windows compiled to PPC was Windows NT 3.5.1. As of Mac OSX 10.6 (Snow Leopard), Apple dropped support for PowerPC. Oddly enough, it has been Apple in the past that has ‘pledged dedication to the platform,’ which, in retrospect, was most likely a ploy to keep people buying PPC machines to clear way for the Intel-based Macs.

Anyway, I’ve gone off topic. I decided that while OS X 10.5.8 is fine for the few things I do on these machines (primarily jailbreaking iPhones — Pwnage tool is so much easier on a mac), I wanted to see which Linux distro’s could compete.

### Installing Linux — the setup for failure




![geeko](http://jpd.ms/wp-content/uploads/migrated/geeko_thumb.jpg)



openSuse was the first I tried, putting it on the Power Mac G5. Novell has also dropped PPC support with their latest version (11.2), so I used 11.1.

Booting up was a mess. I was using a ‘network boot’ cd, which gives you a barebones installer and downloads what it needs from online repos. After realizing that holding ‘C’ to boot from a CD only works with a keyboard that’s _directly_ plugged in to the Mac (not via KVM), I dug up a keyboard and got to work. First boot went into the text-based installer. Setup my network params, pointed it to online repos…nothing. So I tried again — and still, nothing. This happened about ten times. Lucky #11 did the trick. Anyway, I let it pick what it wanted for a ‘typical desktop installation.’ Pretty simple — KDE 4.3, Novell AppArmor, etc.

2.8 GB later, the installation finished. After failing to reboot a couple times, it finally made it back into the setup program to wrap up the configuration. SaX2, the graphics setup program for Suse, ran, found my monitors and supposedly configured everything up.

Another couple failed reboots later, and we’re finally close to ‘the desktop.’ Well sorta, I’m at a standard Linux logon prompt. No X, no KDM, nothing. So I re-ran SaX2, rebooted and got something fresh — a white terminal, and a bunch of odd vertical lines.

The worst part about all of this is that while I was in the installer, the display settings were _perfect._ I tried copying that xorg.conf, but to no avail.

After some work (which included rewriting xorg.conf from scratch, beating my head into the wall, etc), I finally got to K. Of course, the resolutions were all wrong, performance was terrible, and I eventually just gave up.

The final slap in the face came when I told suse to setup yaboot for my Mac partition. The few times yaboot actually loaded, OS X was nowhere to be found.

So long suse.




![jaustin_saturated_full_logo_021_trans](http://jpd.ms/wp-content/uploads/migrated/jaustin_saturated_full_logo_021_trans_thumb.png)



Next I decided I’d try _Volkslinux,_ Ubuntu — linux ‘for the people.’ I’ve used Ubuntu plenty before, so I thought I’d put their no frills Live CD installer to the test. I downloaded the community-maintained 9.10 PPC distro. Rebooted, killed the suse partitions and fired up the installer. Luckily, this time, Ubuntu found my graphics drivers and both of my monitors. I have two DVI monitors — a 25” HF257, and a 23” Samsung. The HF257 goes through DVI → HDMI, and the Samsung goes ADC (one of the dumbest things I’ve seen) –&gt; DVI. I tried to setup the two monitors to be next to each other, but it didn’t work — they were just clones. I figured I’d tinker after it was installed.

Installation was relatively quick (compared to openSuse’s 90 minutes) and painless. Rebooting took me into yaboot, the PPC bootloader. Started Linux from the bootmenu (Ubuntu setup took care of setting up yaboot for OS X on the same partition). Booted into GNOME, all was well. The displays were cloned, but I figured a quick trip to the Displays applet would get that all fixed up. This is where things got wonky.

Ubuntu identified the HF257 as a 23” Samsung, with a max resolution of 1680x1050…while identifying the Samsung as the 25” HF257, with a max resolution of 1920x1080. This is a problem. Xinescape (or whatever the hell they call one large monitor in linux) was working, but with some strange results. No matter what display setup I chose, it defaulted to Monitor 2 on the left, Monitor 1 on the right, not to mention that it still thought each monitor was the other one.

I’ve been working on this some this afternoon, and I’ve yet to find a solution. Since X11 doesn’t really use xorg.conf anymore, it’s been an uphill battle — not to mention that all of the conflicting information on the Ubuntu wiki about which versions of video drivers are included in each version is quite confusing. ATI Catalyst 9.10 drivers are supposed to work for my card, the Radeon 9600, but getting them installed has been anything but easy. Ubuntu doesn’t find them as a ‘restricted driver,’ and documentation is sparse on how to force a refresh of that. Hand-editing the xorg.conf file is a possibility, but a stretch. I may make a living out of screwing around with computers, but dealing with the xorg.conf is a pain in the ass…and that’s what brings me to my next point:

Linux just isn’t ready for prime time. The community has made great strides in getting it to the state it’s in today, but it’s just not there. Imagine explaining the terminal to your dad — or explaining what apt-get means. Imagine his disappointment when he buys a webcam to keep up with his kids at college only to find out that he has to compile a driver from source just to get it to work — or what ‘gcc’ even means.

This core issue is exacerbated by those outspoken members of the Linux community — the ‘ABM (anybody but Microsoft)’ attitude turns off ‘n00bs’ and is extremely juvenile in how it lends support. While there are plenty of supportive forums and message boards out there for new Linux users, there is an equal number of these arrogant, demeaning jerks who make a joke out of anyone who comes for help.

Anyway, this is way too long and I doubt anyone will make it this far, but if you have, thanks!
