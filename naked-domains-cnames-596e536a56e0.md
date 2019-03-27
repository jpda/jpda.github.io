---
title: "Naked Domains & CNAMEs"
date: 2012-10-14T02:48:26.000Z
author: "John Patrick Dandison"

---

### Naked Domains &amp; CNAMEs

If you’ve recently started building on the cloud, you’ll come across a problem pretty quickly — making the root of your domain point to your cloud-hosted service. It’s a real problem, something that has a variety of solutions. But just what _is_ the problem?

### Naked Domains…Sounds Dirty, but really just a PITA

So what is a naked domain, anyway? It’s your domain with nothing in front of it — i.e., johndandison.com. Who types www anymore? Not many people. If you want your app to get traction, it’s gotta hit on the root domain, with nothing in front of it — i.e., the naked domain. A great example of a broken site in action is [belkcredit.com](http://belkcredit.com) — the naked domain does nothing — you must type [www.belkcredit.com](http://www.belkcredit.com) to get to their site.

#### So where does DNS come into play…?

Take a look at the DNS records of a typical website — you’ll find a few records (or a lot, but that doesn’t really matter) — [mxtoolbox.com](http://mxtoolbox.com) is a great tool for doing this. It’s geared towards helping you setup DNS &amp; assorted _accoutrement_, but it is great for checking out DNS.

You’ll find an A record at the root, or _zone apex_ — A records translate names to IPs, so if there’s an A record, that’s the IP someone’s browser is going to try and connect with to get your site. Simple enough, right? CNAMEs also work for translating names to other names, essentially an alias.

#### And this is a problem because…?

Now you’ve got your shiny new cloud service &amp; you’ve been given a DNS name to connect to it. But ‘johns-awesome-blog.cloudapp.net’ or ‘xx-xx-xx-xx.ec2.amazon.net’ isn’t very sexy. In fact, it’s not even useful. How many times have you heard someone say ‘I saw this _stellar_ video on dfw06s07-in-f0.1e100.net (one of youtube’s many CDN/webfarm addresses)! Pull it up so we can watch it!’

So sure, let’s put in an A record of ‘NewAwesomeSite.com,’ pointing to the IP of my EC2 or Azure web instance…

And then it hits. Most cloud service public IPs are only valid for the life of the deployment — so, say you reboot the server…or you publish a new build to staging…now your IP has changed, but your users aren’t getting to your app anymore, until your DNS record has been updated &amp; the TTL has elapsed. Some TTLs can get quite long…you don’t want to be inoperable for hours, do you? Long story short — _don’t use the deployment IP for your cloud service._

#### Ok, well I’ve got CNAMEs, so I can just alias my root, right?

No. It’s not a part of DNS (although word is the DNS group is considering adding it into the spec) — if a record is at the root like that, it’s gotta be an A record. Some providers even let you put them in, but, alas, they just don’t work.

### Ok, so all of this babbling…how do I fix it?

You have a few options. Some are more elegant than others, but they all get you where you need to be.

#### Amazon EC2 Elastic IPs

Amazon has something called Elastic IPs — you can request one (or many) and assign them to your deployments. You don’t get charged for them unless they go unassigned. I’ve used this technique for over a year over at [chattorney.com](http://chattorney.com).

#### Forwarding

A less-than-elegant-but-ok solution is essentially using another web server with a static IP to do 301 or 302 redirects to your service — this is what we’re doing with [moredeets.com](http://moredeets.com) right now. I have cheap hosting for this blog, so I just added a DNS CNAME for www to point to the cloud service &amp; use IIS to do a 301 from moredeets.com to [www.moredeets.com](http://www.moredeets.com). Not pretty, since it relies on external hosting, but it works in a pinch.

#### Services

With three cloud services running at Amazon (and a myriad of different services running in virtual machines), it has become rapidly apparent that I’m not the only one working around this problem. I’ve found a couple of places that can handle it, but they have their own tradeoffs — [DNS Azure](http://dnsazure.com), for instance, requires that you use them as your nameserver. Great for a one-off app, but not so great when you’re running DNS for a business that needs lots of subdomains. CloudFlare also claims to handle this through their EC2 service, but it is more of a website service &amp; I don’t need all of that.

#### Rolling my own. Time to get naked!

I decided it’s time I roll my own. I just need something simple — ping and/or get the IP of the current Azure deployment (unfortunately, the Azure Management services don’t give this data yet, but the Dns namespace in System.Net works well enough), then update a DNS A record with that IP. I’ve moved my new employer on to Amazon’s Route 53 for DNS, so right now that’s the focus — using the Amazon AWS SDK to update the Route 53 A record for your naked domain. Still a WIP, but more to come soon.
