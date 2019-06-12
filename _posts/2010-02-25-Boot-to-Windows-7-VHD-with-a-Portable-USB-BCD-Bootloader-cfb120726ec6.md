---
title: Boot to Windows 7 VHD with a Portable USB BCD Bootloader
description: ''
date: '2010-02-25T21:44:13.000Z'
categories: []
keywords: []
slug: /boot-to-windows-7-vhd-with-a-portable-usb-bcd-bootloader-cfb120726ec6
---

My machine at work has a cracked case, so I had to send it back in for service today. This leaves me without a machine for the next seven business days. I got a loaner T410 from one of the guys which was, unfortunately, loaded with the standard image — Windows XP SP3. I’ve been using Windows 7 now since last summer, so I was less than thrilled to do without Windows 7.

Using disk2vhd from Sysinternals, I took an image of my disk before packing it up. Since I didn’t have time to copy the 70gb image, I copied it over to a Hyper-V server and fired it up.

After fixing the bootloader, I finally got my broken machine’s image running on the Hyper-V server. Unfortunately, the Terminal Services client in Windows XP lacks two things: Support for Network Level Authentication (fixable with some registry keys) and, more importantly, lack of 2D acceleration found in the Windows 7 RDP client.

If only I could install Windows 7 on this loaner box…

This got me thinking…I knew that I could boot from VHD with Windows 7, so I decided to explore that angle. Since the box was running Windows XP, the bootloader was old and didn’t support the new BCD-style bootloader found in NT 6+.

I decided there must be a way to copy the bootloader to a USB key, copy the VHD locally and boot from USB, then into Windows 7.

Here’s what you’ll need.

A Windows 7 machine (not a virtual).

Some Windows 7 media (DVD, bootable USB, etc).

EasyBCD from Neosmart Technologies (the 2.0 Beta is preferable).

A small USB key (I used a 256MB one).

First, format the USB key. I set mine to NTFS — you can do as you so please. Download and install EasyBCD on your Windows 7 box.

When you run EasyBCD, click ‘External Media.’ Point it to your USB drive and do it. This will make your USB key bootable, and drop a blank BCD on it.

### Getting Your VHD

These instructions are for installing Windows 7 to a VHD. If you’ve already done this, then you can skip it. If you’ve already got a VHD, you can skip this too. There are some more thorough instructions (with screenshots) [here](http://blogs.msdn.com/knom/archive/2009/04/07/windows-7-vhd-boot-setup-guideline.aspx).

**Make sure you’re planning to (and licensed to) install Windows 7 Ultimate or Enterprise on your VHD — no other version of Windows 7 supports booting from VHD except these two!**

Next, on your Windows 7 machine, reboot into setup. _It is very important that you do this from your Windows 7 machine — doing it from your Windows XP machine will kill the XP bootloader, and that’s not fun to recover from._

Once setup starts, press Shift+F10. This will open a command prompt.

From here, start diskpart.

In diskpart, type these commands:

create vdisk file="*PUT YER FILE NAME HERE*" maximum="*MAX VHD FILE SIZE*" mode="FIXED or EXPANDABLE"

Hit Enter. Your new vdisk will be created. Now type:

attach vdisk

Now you’ve got your disk. Exit the command prompt, and start setup.

In setup, choose Custom for Installation Type, then pick your disk. You should see your new virtual disk there.

Let the setup start to proceed. At some point, it will ask you to reboot. This is where I did things a little differently.

Rather than letting it reboot back into the setup, I chose to have it reboot back into my existing Windows 7 installation. From here…see the rest below.

### Getting It Running

Get into the OS of your target machine (in my case, Windows XP).

Copy over the VHD you just created from the other machine.

Ok, so you’ve got your VHD locally now. From here, I booted into Windows XP on the target machine, and downloaded EasyBCD.

Fire up EasyBCD. When you do that on XP, it’ll ask you if you want to open an existing BCD store, since there is no BCD on XP. Get your USB key that you formatted earlier, and navigate to \\boot\\bcd.

Once you do this, go to the Add\\Remove Entries screen in EasyBCD. A tab should say Virtual Disk. Click here. Give it a name, and point it to the VHD you copied from your Windows 7 machine.

![image](/img/0_HH8Vr1Q_Sblzpfnh.png)

Save that bad boy and get to rebooting.

### Reboot Process

With your USB key in, boot up off of the USB drive (I have to hit F12 on this Lenovo T410). If you did everything correctly, setup should resume. At this point, kick back and relax as your Windows 7 boots up. If you’re prompted to restart, remember that you must boot from the USB drive to get to Windows 7. Take out the USB drive (or just bypass USB boot) and you’ll get back to an untouched Windows XP installation.

I couldn’t change the system I was using, so this was a good workaround for me.

The shortened version:

* Build BCD database on USB key.
* Make key bootable.
* Create VHD with Windows 7 installation.
* Copy VHD to target machine.
* Add entry to USB-key BCD database for VHD-based Windows 7 install.

Enjoy.
