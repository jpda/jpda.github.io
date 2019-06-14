# Private App Service-to-App Service calls in multitenant PaaS

Recently, [Shannon](https://twitter.com/shankuehn){:target="_blank"} & I got an interesting request from a customer who is using multi-tenant App Services (e.g., non-ASE), but wanted to keep communications restricted to their virtual network. If you're familiar with Azure App Services, typically the way to achieve this is via an [App Service Environment](https://docs.microsoft.com/en-us/azure/app-service/environment/intro){:target="_blank"} (ASE). ASEs are single-tenant, dedicated nodes running the App Service stack _in your virtual network_ - this flips the 'private' bit as they have RFC 1918 IPs from within your VNet. This means you can do pretty much whatever you want with networking - use 17 firewalls in front of it, access vnet or on-prem resources, whatever. Problem is, they can get a bit pricey, at least relative to multi-tenant App Service. Internal ASEs can have a bit of deployment drama too, since you need to manage DNS and get wildcard certificates for your ASE (wildcard certs being particularly difficult to get from security & ops teams). Fortunately, with some (very) recent additions, this is possible using a combination of ['New' VNET Integration (gateway-less)](https://docs.microsoft.com/en-us/azure/app-service/web-sites-integrate-with-vnet){:target="_blank"} and [Service Endpoints](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview){:target="_blank"}/[IP Restrictions](https://docs.microsoft.com/en-us/azure/app-service/app-service-ip-restrictions){:target="_blank"} in Azure. Let's dig in.

## Layout

We've got:

- `app-service-1`, a web app (API) with it's own App Service Plan
- `app-service-2`, another web app (API) with it's own App Service Plan
- a vnet, `vnet`, with three subnets, `app-1`, `app-2` and `default`
- a VM for testing, in the `default` subnet

We need `app-service-1` and `app-service-2` to talk to each other and also be accessible from the VNet, but not from the internet at all. 

## Integrating with the VNet

First we need to get integrated to the VNet from each of our app services. I prefer to keep the app service integrations into their own subnets, makes it easier to segment/separate later. First we need to register the `Microsoft.Web` service endpoint. If you're creating a new VNet, you can do this at creation time. If not, it's easy to enable afterward.
