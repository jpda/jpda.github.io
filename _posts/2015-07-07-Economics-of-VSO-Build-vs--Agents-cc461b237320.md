---
title: Economics of VSO Build vs. Agents
description: ''
date: '2015-07-07T01:56:15.000Z'
categories: []
keywords: []
slug: /economics-of-vso-build-vs-agents-cc461b237320
---

I’m looking into build agents for VSO for a client this week. If you haven’t noticed in your VSO tenant, the build.vNext system is now available in most of them. In fact, it’s not even called build.preview anymore, even though I’m pretty sure it’s still preview. It’s much better than before, now with tasks that are more straightforward and easier to use. No more VS requirement for designing builds, you can do it all in the browser. There’s a lot more, but that’s not really the point of this post.

I found this really by accident — our client needs to use on-premises services that the VSO hosted build controllers won’t have access to, since they’re internal only. This example is a simple ChromeWebDriver for Selenium — it requires Chrome to be installed on the host where Selenium is running, or at a bare minimum, the ChromeWebDriver. They’ve got this on shared hosting internally, but rather than burn time trying to find a standalone chrome executable (let me help you — doesn’t look good), I decided we should do some build agents on an Azure VNet, VPNed back into the private network.

Thinking this was less-than-desirable, since we now have another VM to manage, I started to look at other fringe benefits — namely, cost. What I found _will shock you (thanks, Buzzfeed) — _much to my surprise, it’s significantly different to build your own agents. But first, let’s look at the differences:

#### VSO Hosted Build

![1800COLLECT](https://cdn-images-1.medium.com/max/800/0*DDDojPMvJDmTVN2e.jpg)

The easiest, fastest way to get going with CI or CD builds, or even just builds in general with VSO is to use the hosted build controller. [You get 60 minutes a month for ‘free’](http://azure.microsoft.com/en-us/pricing/details/visual-studio-online/) — I say that because anyone who actually has to buy VSO recognizes that $20/head isn’t particularly cheap, especially when it’s a requirement for backlog access. MSDN subscribers get access for free, so if you’re not a full-blown project management shop and all your people have MSDN, this might not be a big deal. Beyond that, you can buy some more, at a rate of a nickel per minute up until 21 hours, and a penny a minute after that (_why does this feel like an impromptu 1–800-COLLECT ad?_). What do you get for your hard-earned nickel per minute?

*   99.9 SLA for hosted build
*   Virtually zero setup or ongoing maintenance
*   Support for some third-party unit test frameworks
*   Visual Studio 2015–2010 preloaded on the environment

However, there are some restrictions — being multi-tenant, this is no surprise:

*   no builds taking over 10 hours to run
*   no builds over 10GB
*   no admin privileges (this is obvious, and a bad practice anyway, but everyone’s got some legacy skeletons in their closets)
*   no interactive or user-logon dependencies

So for most things, the hosted build controller is fine. Pre-configured, one-click CD build definitions for integrating with your Azure resources too.

#### 1–800–4DEVOPS

This is all great, until you start thinking about modifying your deployment process to include some rapid releases. Let’s say you’re on a small team of four developers, checking in code at least once a day. Each of those check-ins triggers a CD build, and you’ve got one overnight for good measure. Let’s also say it’s a relatively simple project that takes around 5 minutes to build. You can get through about a day and a half with your included build minutes.

“No problem,” you say, “I’ll just buy the minutes I need from VSO.” Let’s extrapolate that a bit — five builds per day @ 5 minutes/build. 25 minutes a day, five days a week = 125 minutes/week. Typical 22 day work-month, that’s around 550 build minutes/month. We’ll round to 600 for giggles.

600 minutes, minus your 60 free minutes leaves you holding the bag on 540 build minutes. At $0.05/min, that comes to $27. What else does $27 buy you?

*   182 hours of a Basic A2 VM (2 core, 3.5GB)
*   91 hours of a Basic A3 VM (4 core, 7GB)
*   A pack of smokes in NYC
*   77% of a Raspberry Pi 2

182 hours is a quarter of the Azure-standardized 744 hour month. So you could host your own build agent, leave it running most of each week and unleash quite a bit more power to get builds done faster. Or, in our case, you could host it on a VPNed VNet and connect to on-premises resources. Or join it to a domain and run the build agent service as a local administrator. Or execute powershell that requires admin rights. Or…

What would be even better is a way to trigger that machine to start-up via Azure SM/PowerShell only when a build is queued, but I haven’t thought that all the way through yet. Nailing that would add even more efficiency.

Now I know this doesn’t take into account things like the additional maintenance of yet-another-server, or the (minor) addition of storage + egress cost, or even the fact that a single VM in Azure doesn’t have the same 99.9% SLA. But even then, it’s a path worth exploring, especially if your builds are scheduled so you can automate that agent coming up/down, or if you fall outside of the prescribed requirements for using the hosted controller. With MSDN images that already have Visual Studio or TFS installed, and the fact that the build agent ‘installer’ is just a powershell script, those machines could be created from scratch with minimal effort. Resource Manager + PowerShell DSC + Startup Script might just be the ticket to a super-fast and easy build agent with only slightly more overhead than using the hosted controller.