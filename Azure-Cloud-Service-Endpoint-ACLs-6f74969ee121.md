---
title: "Azure Cloud Service Endpoint ACLs"
date: 2014-06-10T19:16:09.000Z
author: "John Patrick Dandison"

---

Recently, Azure VMs got endpoint ACLs — this is a great addition and one of the biggest things I missed from AWS’ security groups. Using them on VMs is great and all, but what about cloud services? Since VMs are instances within a cloud service, it’s certainly _possible,_ but how can we configure them as such? Fortunately it’s pretty easy.

#### No soup

First you’ll need to snag Azure SDK v2.3 and make sure your ServiceConfiguration.&lt;env&gt;.cscfg is at the latest schema (as of today, that’s 2014–01.2.3).

Head on in to ServiceConfiguration.Cloud.cscfg — these restrictions are obviously cloud-only — and add your chunk of config. Intellisense should pick this up and make it much simpler.

What’s nice is you can define your rules in total under AccessControls, then assign them as you need them to specific endpoints.

Here’s a sample allowing a few single IPs + a range and denying everyone else. These are executed in order, so be aware of the order tag.
> &lt;NetworkConfiguration&gt;  
> &lt;AccessControls&gt;  
> &lt;AccessControl name=”DenyAllExceptDevelopment”&gt;  
> &lt;Rule action=”permit” description=”stuff” order=”100&#34; remoteSubnet=”198.51.100.194/32&#34; /&gt;  
> &lt;Rule action=”permit” description=”thing” order=”101&#34; remoteSubnet=”192.0.2.167/32&#34;/&gt;  
> &lt;Rule action=”permit” description=”biz” order=”106&#34; remoteSubnet=”203.0.113.0/24&#34;/&gt;  
> &lt;Rule action=”deny” description=”theinternet” order=”200&#34; remoteSubnet=”0.0.0.0/0&#34; /&gt;  
> &lt;/AccessControl&gt;  
> &lt;/AccessControls&gt;  
> &lt;EndpointAcls&gt;  
> &lt;EndpointAcl role=”AzureService.Thing.Stuff” endPoint=”Endpoint1&#34; accessControl=”DenyAllExceptDevelopment” /&gt;  
> &lt;/EndpointAcls&gt;  
> &lt;/NetworkConfiguration&gt;
