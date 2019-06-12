---
title: Windows 10 IoT — Device Provisioning + App Deployment
description: ''
date: '2015-09-24T22:16:39.000Z'
categories: []
keywords: []
slug: /windows-10-iot-device-provisioning-app-deployment-eedcdb1ac90a
---

Windows 10 IoT — the promise of a universal app that can run, quite literally, anywhere. This includes tiny, cheap computers like the Raspberry Pi and Minnowboard MAX.

![Raspberry Pi 2Raspberry Pi 2](/img/0_7zmZHbLexlqTHVPo.jpg)
Raspberry Pi 2

It’s quite neat — $35 gets you, effectively, a tiny PC. Perfect for running a [media center](https://osmc.tv/), a [web server](https://www.bing.com/search?q=rpi%20webserver), arcade machines, all sorts of fun stuff. The advent of a specialized Windows 10 build being available opened the doors to the massive base of .net developers who, up until now, could struggle with Mono on Linux or have to learn a moon language like Java. On July 29th, an ‘RTM’ build dropped for Windows 10 IoT — plus Microsoft has been spending a lot of time hyping up the IoT Suite, which is basically a collection of existing products (Stream Analytics, Event Hubs, etc) that are getting a bundled offer this fall. Also coming this fall, Cardinal’s annual Innovation Summit — a place where we can meet with customers new and old to talk about cool stuff we’re building and how it fits into our customers’ daily lives. This got me thinking — what can we do to showcase the power of Windows 10 IoT + Azure IoT?

(Shameless plug — I’ll be talking about this and showing some cool demos [10/8 in Charlotte](http://www.cardinalsolutions.com/MISCharlotte) and [10/14 in Columbus, OH](http://www.cardinalsolutions.com/MISColumbus) — follow the links to get registered — it’s free — and come check it out)

#### The Plan

I managed to get my hands on about a dozen Raspberry Pis — for the Summit talk, that would be plenty. I won’t get into details about _what_ I’m building in this post — you could always, you know, [come to the Summit](http://www.cardinalsolutions.com/MISCharlotte) and see it in action — but I will say it involves bluetooth and nearly the entire Azure IoT suite…but for this post, I’ll talk mostly about my biggest pain points — provisioning and app deployment.

![Pi explosion.Pi explosion](/img/0_fpv3knRWh6VUWHcE.png)
Pi explosion.

#### Provisioning

What’s the first thing we need to do? Get these things installed + provisioned. Installation is relatively simple, albeit time consuming and manual. It involves imaging each SD card individually, which usually takes about 10 minutes. We had a couple laptops going and managed to get them provisioned pretty quickly. Once they’re installed, they’re all ‘minwinpc,’ which isn’t terribly helpful for discerning one from another. Not to mention the hard-coded, out-of-the-box admin password ‘pass@word1’ isn’t terribly secure. We need to change the names + update the passwords, preferably in a scriptable way. We’ll start there.

I’m not going to get into the gory details of setting up remote Powershell access, you can [read all about that here](http://ms-iot.github.io/content/en-US/win10/samples/PowerShell.htm). We’re just going to extrapolate that out a bit. Take a look:

{% gist 375707054983fbdb429e %}

Pretty straightforward. Not a lot happening there — you could wrap this in a foreach loop and call it for each device you’ve got on your network.

#### App Deployment

We’ve written this nice Windows 10 Universal app and it’s time to deploy! Now what? …This is a long one. And irritating. Every official bit of guidance says (usually with too much excitement), ‘Deploy right from Visual Studio!’ — which is \*great\* for debugging…and completely horrible for deploying to more than one device. This is the internet of \*things\*, Microsoft. Not the internet of \*thing\* — but let’s look at that official guidance:

* Open project properties
* Choose Build
* Change to Remote Machine
* Type in name of remote machine, or grab the mouse (\*ew\*) and choose from the list
* Right-click project, Deploy…
* Deploy dependencies
* Deploy appx

Which is great if you paid by the click, I suppose. For the rest of us it’s miserable. That’s just \*one\* app deployment to \*one\* device. And it requires Visual Studio! Tell your deployment teams they all need VS licenses for deployment…I’ll wait… Didn’t go so well, eh?

#### Web-based Deployment

Windows 10 IoT comes with a simple web portal (http://<your-pi>:8080/) for info + tasks — task/process monitor, event logs, etc. Plus app deployment. This should be easy, right? No. Apparently there’s some wonkiness with the order of operations + existing dependencies here, where I could get this to work exactly zero times. But we’ll be back…

![AppX Manager from WebBAppX Manager from WebB](/img/0__Glo4ohc4O1osRwS.png)
AppX Manager from WebB

#### Modern Windows App Deployment, aka, WinAppDeployCmd

[WinAppDeployCmd](https://msdn.microsoft.com/en-us/library/mt203806.aspx) — this sounds perfect, right? Should do \*exactly\* what I want, which is to **Deploy** a **WinApp** from a **C**o**m**man**d** prompt. Perfect! No. This doesn’t work. You can’t get a PIN like you can from a phone to allow remote sideloading. Don’t even waste your time with this.

#### Modern Windows App Test Deploy, aka Install-AppDevPackage.ps1

If you’ve done Windows 8+ modern app dev, this should be familiar. I know, we’ll just copy the bits, including the ps1, and deploy from that using remote powershell. What could go wrong? **Everything.** What does this script _do,_ anyway? It’s pretty simple, really:

* Checks for a developer certificate (which isn’t required in Windows 10) for ‘Developer Mode’
* Installs the cert, if necessary
* Attempts to install the app package + signing cert

RPis use ARM chips — so tools like certutil don’t exist. Know what uses certutil a lot? Install-AppDevPackage.ps1. I yanked every bit of reference to dev certificates out, seeing as there’s just a quick registry change for ‘Developer Mode’ in Windows 10. It all eventually died at Add-AppPackage — RPC endpoint mapper was out of endpoints (uh, endpoint mapper, isn’t this your only job?). I figure it may be related to permissions more than anything — apps are deployed + run under an account called DefaultAccount — not Administrator or whatever user you’ve connected as.

> **PSA**: don’t change the password of DefaultAccount. This should go without saying, but there’s nothing to stop you and it’ll require a reimage. It was late when I tried this.

Maybe someone else can make this work, but I eventually abandoned it, since I had to reimage my device.

#### devenv.exe /deploy /target:ARM

In a rare stroke of brilliance, I thought — I know, I’ll just do whatever Visual Studio is doing, just over and over again, through the command line! This, also, did not work. Why, you ask? Because the ‘deploy’ option through Visual Studio uses some bastardization of the remote debugger (msvsmon.exe), with a private, undocumented service that it pumps data through. Visual Studio’s response trying to deploy?

> Remote debugging is not available from the command line.

Yes. _Evil._

#### TailoredDeploy.exe

Tracing the logs and deployment, I had a couple more leads (note — MSBuild ‘diagnostic’ debugging is no joke). Notably, TailoredDeploy.exe. I burned a few hours trying to figure this out, to no avail. The apparently useless endpoint mapper still couldn’t do its job, leading me to believe all of these higher-level tools really call the same methods underneath.

#### Revisiting Web Deployment

At this point, I had spent one too many late nights beating my face on the keyboard, and likely 10x the amount of time it would take to have just deployed manually. In a truly last-ditch effort, I ventured into the MSDN Forums. Here, I was pushed in the direction of a REST API that existed and was hosted by WebB (the little web server) on the Pi. After waking up from the stunning surprise of someone actually contributing a useful answer in the MSDN Forums, I got to fiddling and finally had something real to chase. It also unlocked some insight into _how_ apps are deployed onto these little devices. It wasn’t without its own frustrations, however.

#### /RestDocumentation.htm

So innocuous. So simple. So _obvious_. Surely this little web server used something to perform its actions — I couldn’t believe I hadn’t thought of this before. Head over to your web browser and hit /RestDocumentation.htm off your Pi — you’ll find a list of interesting things you can do via HTTP with your Pi. Notably, App Deployment. Following these ‘docs’ to the T (there’s not much there), I was still getting 400s and 500s, although it wasn’t immediately obvious why. I decided that rather than trying this the ‘proper’ way, it was time to pull out the stops and just fiddle the hell out of it and figure out what the web interface was doing. Surely they didn’t write this logic twice…? ‘But you said it didn’t work above,’ you might be saying. And this would be true, it never did work from the web interface, although I never spent a whole lot of time trying to figure out why. Through a bit of trial and error, I figured out that the dependencies didn’t seem to be needed; in fact, including them caused it to blow up. Just uploading the package + certificate seemed to work fine, even on a freshly reimaged device. Fiddler showed me a few interesting things in the process:

![Twiddlin’ bits in Fiddler…or would it be Fiddlin?fiddler](/img/0_AVcn-sWM1sViyQx2.png)
Twiddlin’ bits in Fiddler…or would it be Fiddlin?

* Post the form data (e.g., the packages)
* Post dependencies
* Post certificate
* Commit deployment

There were a few other things I wanted to do too — like uninstall the app before it gets reinstalled (to start with a fresh slate, mostly because I was getting a lot of ‘file in use’ errors) and set my app to start at boot, as would be typical for a production deployment of an IoT app. Take a close look at the above — you’ll see a lot of paths starting with /api/appx — but a few are /api/iot/appx — there’s a difference here. One is the seemingly ‘private’ API used by the web interface. There is no documentation I could find about these, but I definitely needed them, especially for actions like setting default boot apps. I would guess there’s some sort of privilege boundary here, but I can’t say for sure. Fiddler trace in hand, I got to replicating this in Powershell. You may laugh, as I would at someone who says that, but I figured I need to get my PS chops better…and PS is really just C# with a syntax from the moon, so what’s the big deal? _How hard could it be?_

#### Take 1

I got to work, plunking out some HTTP requests with Powershell. This went well, until I started uploading the packages. For whatever reason, it would 500 immediately, with no logging to indicate the problem. When I took the request and reissued it in fiddler, it would fail. But when I took it into the composer and reissued it, suddenly it would work. I went back and forth for hours, comparing requests, finding nothing seemingly different between the two with the exception of the multi-part form boundary ID. I even got to the point of reflashing one of the devices to make sure I didn’t have any sort of previous-deployment hangover that was preventing the upload. Still nothing. At this point I was grasping at straws. Tired, frustrated. Ready to launch the Pi from the nearest potato gun. On further inspection of the two requests, I found _one_ thing that was different between the two — the one on the left returns 200, the one on the right returns 500. See if you can find it:

![Can you find it?srsly](/img/0_RwQkQzyJWegzSTLt.png)
Can you find it?

#### Let’s talk about RFC 2616 19.2 — multipart

Specifically, let’s talk about multipart boundary identifiers. From the [spec:](http://www.ietf.org/rfc/rfc2616.txt)

> Although RFC 2046 \[40\] permits the boundary string to be quoted, some existing implementations handle a quoted boundary string incorrectly.

I think we’ve found one of those implementations. But there is a larger issue here — .net’s inconsistency with how it applies these headers. Why would the boundary be in quotes in the declaration but naked in the usage? I understand that per spec, web servers _should_ accept either as the same, but it’s just opening the door for trouble when the standard implementation rolls the dice on _every request._ What else have we learned? The little WebB.exe web server in Windows 10 IoT doesn’t implement the spec properly either — it should _only_ be looking for a CRLF and two dashes for the boundary ID, not checking explicitly. Let’s get to the code. You’ll see an explicit drop/re-add of the **Content-Type** header with my boundary ID so it matches; it’s really about the best workaround I could find.

#### Multi-Deployment PowerShell

{% gist 5c114b83e229075ed062 %}

#### What have we learned?

![This is not some mundane detail, Michael!mundane details](/img/0_wA8fWYNNpZD1w8uq.png)
This is not some mundane detail, Michael!
