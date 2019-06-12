---
title: Let’s talk about securing web apps with Azure AD
description: >-
  Here’s a common scenario a lot of our customers hit early in their cloud
  journey — especially when dealing with enterprise customers using…
date: ''
categories: []
keywords: []
slug: ''
---

Here’s a common scenario a lot of our customers hit early in their cloud journey — especially when dealing with enterprise customers using AD + IIS in their web stack:

> I’ve got lots of LOB apps that \*just work\* with Kerberos and Windows Integrated Authentication in IIS. How do I move this app to the cloud? How do I get this app into PaaS services like Azure Web Apps?

### The Identity Landscape

But first, let’s talk briefly about the landscape. In our traditional, on-prem, AD, IIS and Windows stack, we’re relying on Kerberos or NTLM for a lot of auth. We turn on Windows Integrated Authentication on our IIS Web App, maybe change the AppPoolIdentity to a domain account so it can connect to SQL using that domain account. And our users are on Windows PCs, using IE. Right?

You, as an app developer, may be thinking — why does this matter to me? I write code, I throw it over the wall to the infrastructure/deployment folks, and they make magic happen. All I need to know is to use **RequestContext/HttpContext.Current.User/OwinContext/Principal.Identity**, right? 

What I’ve found over years of development in various shops is that developers (particularly enterprise developers, although this may just be a function of my travels) in Windows environments really don’t spend much time worrying about identity, mostly because a lot of people have spent a lot of time, money and effort making it simple to use. 

Let’s look at everything it takes to make **User.Identity.Name** populate with _contoso\\john_— it may be more than you think:

First, we need a directory. In our case, it’s Active Directory, which means setting up a new server, installing AD and configuring a domain. If we want Kerberos, we need a KDC too. Inevitably, we’ll need two of these just in case one decides to stop working. We’ll need time and DNS also running, since Kerberos is dependent on those things. 

Next we need a database server, most likely running SQL Server. And no SQL Authentication, because our security policy says no. So we’ll need to create a new server, join it to the domain and install SQL Server. We’ll probably need to go ahead and create some SQL Service accounts in AD too, to run the service. 

Next we need a server to host our web app. Which means a new server, joined to the domain, IIS installed. Let’s create some IIS AppPool service accounts in AD too, since we’ll need them to connect to SQL, since SQL is only set to use Windows Authentication. 

Oh and don’t forget your SPNs, especially if you’re doing a SQL Cluster or something similar. Or using a different alias for connecting to your web sites in IIS. 

Oh, and lastly, let’s not forget that your users will need to be using a domain-joined machine too. And a Kerberos/NTLM-aware browser.

You know what can’t join a domain? Mobile devices. IoT devices. Chances are you’re reading this on an iPhone or something similar. Or a tablet. Or something _not_ domain-joined.

You know what that sounds like? Sounds like a lot of infrastructure to me. Also sounds like a lot of client restrictions and ‘things in the right place.’

The net of this is we end up with two layers of trust — a device trust (via domain-join) and a user trust (via the user’s login). Problem is, anything that relies on a device trust is going to be onerous to manage without centralized management. Also means it’ll be difficult to meet your users where they are — off-network and on mobile devices.

### Too reliant on infrastructure

If we can centralize our trust on the identity, we can solve many problems in this equation. Let’s review what we outlined above:

Infrastructure: AD DCs, Web servers, SPNs, DNS, Time, Network connectivity/VPN. Development: **User.Identity.Name.**

That feels a bit out of balance. It also means I need a staff to deploy and manage all of this, plus a host of licenses to 

As we move to cloud, the biggest benefit comes from PaaS services — this is where we can get much closer to the ‘pay for what you use’ model vs. the ‘pay for what you provision.’ We can deploy a VM and size it appropriately, but there’s so much management involved in that, you’re really just moving from one hypervisor to another. But PaaS services are elastic, shared and significantly more efficient and optimized than doing it yourself.