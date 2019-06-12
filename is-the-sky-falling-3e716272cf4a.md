---
title: Is the sky falling?
description: ''
date: '2015-03-25T01:55:23.000Z'
categories: []
keywords: []
slug: /@jpda/is-the-sky-falling-3e716272cf4a
---

Today [was a neat day in the Azure space](http://weblogs.asp.net/scottgu/announcing-the-new-azure-app-service) — Azure Websites has grown up and found itself. We’ve got new units of functionality that can build fully functional apps and workflows, interacting with different systems and moving data around (e.g., BizTalk), through a _designer_ on a web page! Amazing. I came here tonight to dig in and share my thoughts on the new services, but I got sidetracked.

After the announcement, I kept up with social networks, internal and external, and generally there’s a healthy level of excitement. I think once people get their hands dirty, we’ll see a lot more excitement — but what I also saw was sadly typical when these kinds of announcements are made:

> What makes me valuable is now available for anyone who can drag-and-drop on a webpage.

And this assertion would be correct, except for one crucial detail — what makes us valuable as software developers, engineers, architects, code monkeys, etc is everything _except_ physically typing out the code. If your only value is in the physical delivery of the code, then it may be too late for you anyway. Let’s back up though.

#### Engineering + Heavy Clouds

Look at systems engineering over the past 10 years or so. These poor souls have had all kinds of the-sky-is-falling moments. First it was virtualization, then the cloud. Then SaaS — Office 365, SharePoint Online, Exchange. If your job involved managing and monitoring servers and services for your company, your job has been under attack for a decade..

But has it? How many people lost their jobs because their company elected to deploy Office 365? Many people adapted existing skills and learned new ones. I’ve yet to see “WILL ADMINISTER SHAREPOINT FOR FOOD” signs littering once-vibrant office parks. I once read that _if change isn’t something you’re interested in, technology is_ **_not_** _the industry for you._ That statement pretty much summarizes the majority of this post, so feel free to leave now if you’ve gotten what you came for.

In all reality, jobs in the space have stayed relatively stable in relation to other IT jobs. For example, if you look at the trends over the past 10 years, systems administration and software engineering jobs have followed a similar course:

![Software Engineer — source: indeed.com/jobtrendsSoftware Engineer - source: indeed.com/jobtrends](https://cdn-images-1.medium.com/max/800/0*EBUXlEjssojnM5xl.png)
Software Engineer — source: indeed.com/jobtrends![Systems Administrator — source: indeed.com/jobtrendsSystems Administrator - source: indeed.com/jobtrends](https://cdn-images-1.medium.com/max/800/0*YR8yS6hQQ8EPOO_X.png)
Systems Administrator — source: indeed.com/jobtrends

See how similar they are? People aren’t being replaced — in fact, these graphs are a little disingenuous as the ‘overall percentage of matching job postings’ includes most job posts on the internet, which are, of course, exploding. The point, however, is that we’re seeing the same general trends in both systems and software engineering. What did people do? They adapted, they translated existing skills into new platforms, they learned new chunks of knowledge to handle what was coming their way.

Why? Think about it — _on what hardware_ Exchange is installed on is irrelevant. The hardware is commodity now. Administering Exchange requires a certain set of skills; before Office 365 and after those skills aren’t dramatically different. Sure, it’s fewer servers to manage, but how many Exchange admins were really managing that enormous Jet database manually anyway? That knowledge and skillset transfers readily.

#### Software Development Is Next.

There have been pivotal changes in software in the past ten years — virtualization to an extent and (obviously) cloud. Maximizing efficiency of resources and time-to-market agility has made the cloud what it is. We’re in the ‘coarse’ efficiency now — the next five years or so will bring a whole new era of abstraction and efficiency.

Anyway — let’s get back to my original issue. Software engineering is already going through some significant changes, but one of the biggest ones speaks directly to my original issue above. At some point, skills become commodity. Is there anyone working in dev today that can’t readily find sample code for connecting to Salesforce.com/Dropbox/Office 365 and make that work for their application? It’s become so commonplace that that’s no longer a ‘special’ skill — in fact, it has been made so **repeatable** that we can drag a block with an Office logo on it and connect to SharePoint Online data without writing _any_ code.

Who out there is impressed that I wrote a web app that had a nifty sliding progress bar? Anyone? Bueller? Bueller? That’s not impressive anymore. Years ago, when XMLHttpRequest was new, making a web call _without a postback_ was _amazing._ Mind = blown. Now there are dozens of frameworks that make many, many lines of code boil down to a single line:

$(“thing”).progressBar();

Are you going to put ‘implemented progressBar’ on your resume? It can sit right next to ‘managed to get both feet into shoes.’ I think not.

#### Platform Dev vs. Implementers

It’s silly to think that what we’re going through now is somehow different from what we’ve gone through forever. But there is, among all the change, one constant that seems to be creating a larger gap daily. Platform development and implementation.

Take a look at what was announced in the Azure space today — ‘Logic Apps,’ ‘API apps,’ — each one a higher-level abstraction of a few existing pieces that let you compose services from existing building blocks. The guys building those blocks have no idea what you’re going to do with them and in what combinations you may elect to build. But it doesn’t matter. The software is written in a way that supports, nay, _encourages_ that kind of development. If I can get away with dragging seven existing blocks onto a designer and solve whatever problem I was attempting to solve, how is that not _stuffed to the gills_ with win?

Better yet, say there’s not a block that does what I need. Let’s build the block _and_ write it properly so _other people_ can use the block. Sounds pretty neat, huh?

#### Which are you?

When you start a project, do you write a bunch of problem-specific code? Are you one-offing everything you do? How much code reuse is in that block that you just wrote? Your time becomes _less_ valuable for busy work when someone else can implement someone else’s blocks + 10% new code in half the time. If you’re solving a problem, solve it once and use it as many times as possible. Microservices and distributed components are how you gain maximum leverage for the time you’ve already spent.

#### Platform Dev is the Future

This should be obvious if you’ve made it this far, but I think it’s fairly clear that platform development is where the world of software development is heading. That doesn’t mean custom software won’t exist, but it won’t be ‘built from the ground up.’ It’ll be built from existing blocks of increasingly complex, reusable code. Conceptually this isn’t different from the frameworks we use today. When was the last time you managed memory? Opened raw sockets and sent HTTP requests manually? All of these things are offered by most of the major players today, to abstract complexity and menial, repeatable tasks. As we’re seeing today, the API/reusable block market is exploding. If that means your job is in danger, then perhaps it’s time to start thinking platforms and stop writing code merely for the finger exercise.

Always think about a platform, always think about how you can make your code as generic and reusable as possible, and think about what kinds of other uses it may have. Build for platforms, not for implementations.