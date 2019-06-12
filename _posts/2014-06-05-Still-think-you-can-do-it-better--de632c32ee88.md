---
title: Still think you can do it better?
description: ''
date: '2014-06-05T18:03:13.000Z'
categories: []
keywords: []
slug: /still-think-you-can-do-it-better-de632c32ee88
---

My wife’s work laptop is a joke. Although she has no administrative rights, it recently got infected with one of those ransomware-type viruses. I tried to help her out — what I found was pretty awful.

She works for a bank. A _bank._ Where _people keep money._

* Antivirus? Bah. AVG Free. Pretty sure that’s a gross license violation, not to mention it’s just so incredibly _bush league._
* AVG’s ‘safe search’ set as the homepage and locked to prevent changes. Interestingly enough, these all have a specific client ID attached to the URL. I almost wonder if they are getting search kickbacks…
* Windows Updates only through WSUS.

WSUS isn’t bad on its own, but when it’s only available on-prem (or through VPN), you leave disconnected workers (like her) out of the patching process. An arguably minor violation, just connect to VPN and away we go.

That is, of course, provided someone is managing your WSUS.

I took these screenshots, undoctored, last night, 6/4/14.

Most recent check: 1/12/14.

Last installed: **8/18/12.**

![omg](/img/0_UDQCWfpv5Q9hQHLn.png)

But then it got really awful.

No updates available.

![omg2](/img/0_UyLLAAXBSksCxH3S.png)

Let’s review — you can’t get updates from anyone except for your employer, but your employer obviously has people managing your systems who aren’t capable. It’s another reason that people, humans, end up getting in the way, be it hubris or ignorance. It’s dangerous, and in all honesty, Microsoft shouldn’t allow this to continue. There really should be some kind of dead man’s switch to allow people to get updates when a WSUS operator has effectively gone dead.

It’s really more of an indicator as to how bad things really are in the corporate landscape. Legions of people who mistakenly believe that the people they’ve hired to manage their infrastructure are somehow more capable than people who run massively scaled services for thousands (or millions) of customers daily. That’s not to say there aren’t bad eggs in the cloud space or diamonds on-prem, but those are exceptions to the rule. IT Managers and execs who see the cloud as a threat to their budget or headcount need to reevaluate — would you trust your personal information with these people? Would you trust your family’s livelihood with these people?

If the answer is no, it’s time to take a second look.
