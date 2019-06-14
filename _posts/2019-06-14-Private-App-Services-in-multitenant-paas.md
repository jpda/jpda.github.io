---
title: Private App Service-to-App Service calls in multitenant PaaS
description: ''
date: '2019-06-14T18:11:10.1901448Z'
categories: ['azure', 'app service', 'networking']
keywords: ['azure', 'app-service', 'ase', 'vnet-integration', 'service-endpoints']
slug: /pesudo-ase
---

Recently, [Shannon](https://twitter.com/shankuehn){:target="_blank"} & I got an interesting request from a customer who is using multi-tenant App Services (e.g., non-ASE), but wanted to keep communications restricted to their virtual network. If you're familiar with Azure App Services, typically the way to achieve this is via an [App Service Environment](https://docs.microsoft.com/en-us/azure/app-service/environment/intro){:target="_blank"} (ASE). ASEs are single-tenant, dedicated nodes running the App Service stack _in your virtual network_ - this flips the 'private' bit as they have RFC 1918 IPs from within your VNet. This means you can do pretty much whatever you want with networking - use 17 firewalls in front of it, access vnet or on-prem resources, whatever. Problem is, they can get a bit pricey, at least relative to multi-tenant App Service. Internal ASEs can have a bit of deployment drama too, since you need to manage DNS and get wildcard certificates for your ASE (wildcard certs being particularly difficult to get from security & ops teams). Fortunately, with some (very) recent additions, this is possible using a combination of ['New' VNET Integration (gateway-less)](https://docs.microsoft.com/en-us/azure/app-service/web-sites-integrate-with-vnet){:target="_blank"} and [Service Endpoints](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview){:target="_blank"}/[IP Restrictions](https://docs.microsoft.com/en-us/azure/app-service/app-service-ip-restrictions){:target="_blank"} in Azure. Let's dig in.

## Layout

We've got:

- `jpd-app-1`, a web app (API) with it's own App Service Plan
- `jpd-app-2`, another web app (API) with it's own App Service Plan
- a vnet, `vnet1`, with three subnets, `jpd-app-1-subnet`, `jpd-app-2-subnet` and `default`
- a VM for testing, in the `default` subnet
- The two apps & vnet live in the same region

We need `jpd-app-1` and `jpd-app-2` to talk to each other and also be accessible from the VNet, but not from the internet at all.

## Integrating with the VNet

First we need to get integrated to the VNet from each of our app services. I prefer to keep the app service integrations into their own subnets, makes it easier to segment/separate later. First we need to register the `Microsoft.Web` service endpoint. If you're creating a new VNet, you can do this at creation time. If not, it's easy to enable afterward.

During creation:

![service endpoints](img/Annotation 2019-06-14 141341.png)

After creation (from the VNet --> Service Endpoints pane). Enable it on all the subnets you want to access our App Services from.

![service endpoints](img/vnet-service-endpoints.png)

Once you've enabled them, head to the Networking pane of the first App Service (`jpd-app-1` in my example).

![networking pane](img/jpd-app-1-networking.png)

Configure VNet Integration, then choose Add VNet (Preview). Choose your vnet, then your subnet.

![add feature](img/jpda-app-1-networking-add.png)

Do this for both of your App Services, integrating each into their respective subnets. At this point, our app services are now connected to `vnet1` - if we had resources we needed to access in the VNet, like a VM, a SQL Managed Instance or even something on-prem via site-to-site VPN, we'd be able to by this point. The app services are exposed to the internet, however. We've enabled access _from_ the App Services to the VNet, but we haven't restricted access _to_ the App Services to only the VNet just yet. For that we'll use Access Restrictions.

## Access Restrictions

Next we need to enforce the access restrictions. Back in the networking pane of your App Service, you'll see 'Access Restrictions.' This is where we'll add our vnets. By default, your App Service will allow all traffic from all sources. You can leave the default Allow All rule, since as soon as we add our own restrictions, the Allow All rule becomes a Deny All, with our rules taking priority. 

You'll also note there are _two_ hosts listed - `<yourapp>.azurewebsites.net` and `<yourapp>.scm.azurewebsites.net` - the first one is your app, the second one (*.scm.*) is the backstage view of your web app, the Kudu console. You can restrict these independently. Make sure you add restrictions for both the main site and Kudu/scm site if you don't want any access from the internet!

![sites](img/jpd-app-1-access-kudu.png)

Now let's add our restriction! Since this is `jpd-app-1` and it's integrated into the `jpd-app-1-subnet` subnet, we need to add two rules:

- Access from `jpd-app-2-subnet` subnet
- Access from `default` subnet
- Deny everything else

First the `jpd-app-2-subnet` rule. Make sure you choose 'Virtual Network' in the type. Then choose your vnet (`vnet1`) and your subnet for the other app that needs access (`jpd-app-2-subnet`). Repeat for (`default`) to enable the rest of the vnet and any other vnets/subnets that may need access.

![app2](img/jpd-app-1-add-app-2.png)

## Repeat for the other app

In your other app (`jpd-app-2`), do the same thing. Add access restrictions, only this time choose the subnet of the other app (`jpd-app-1-subnet`). 

## Testing

If you go to your app in a browser from your local machine, you should get a 403, 'web site stopped.' This is the experience for app services that are restricted - it's a 403 Forbidden, stopped is a bit misleading here. 

![stopped](img/jpd-app-1-stopped.png)

From your web app's kudu console (`<yourapp>.scm.azurewebsites.net`), which is either accessible only from the VM on your vnet, or from the internet, if you didn't add a restriction, we can do a curl to the other app to make sure our configuration is all square. `tcpping` isn't valid here, because the site does respond to ping - as it is available on the internet, but returning `403`.

`curl -sI https://<yourapp>.azurewebsites.net`

![app-1-curl](img/jpd-app-1-curl.png)

This should return some HTML/markup. If it's a `403`, something is wrong. If you get a `200/OK`, you're in business!

![app-1-to-app-2](img/jpd-app-1-to-2-success.png)

_Note: you may notice that my URLs changed near the end - I added a bad rule so I had to trash the two app services and recreate them. **Don't put zero (0) as a priority for a rule!**_

I haven't tried this with Functions on a consumption plan yet, but that's coming next. Stay tuned!
