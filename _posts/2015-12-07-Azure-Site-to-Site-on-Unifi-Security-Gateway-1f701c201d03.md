---
title: Azure Site-to-Site on Unifi Security Gateway
description: ''
date: '2015-12-07T02:25:27.000Z'
categories: []
keywords: []
slug: /azure-site-to-site-on-unifi-security-gateway-1f701c201d03
---

![It can be done. A little painful, but doable.It can be done. A little painful but not bad.](/img/0_7ulwBkZt6e6CXP81.jpg)
It can be done. A little painful, but doable.

Recently decided to upgrade from my Netgear SRX5308 here at home to a shiny new [Unifi Security Gateway (v3)](https://www.ubnt.com/unifi-switching-routing/usg/). Quieter, much less power consumption, but most importantly, higher WAN-to-LAN throughput than the Netgear, which his a requirement now that ATT and Google have brought 1Gbps+ fiber to Charlotte.

I’ve used the Unifi APs now for years and loved them, the AC access points are particularly good. Blazing fast and great coverage throughout the house with only two of them. Dozens of 2.4GHz networks nearby, but 5GHz is pretty much completely empty. I was on the lookout for more Unifi gear and the USG caught my eye. What seems like a repackaged EdgeMAX Router has one big difference — instead of an onboard web server and config UI, all the config is done by the central Unifi Controller. I like this, since I’m already using it for my APs, and figured more integration would be neat.

I was wrong. To say the least, this was a huge PITA. The UI has no support for site-to-site VPN configuration (or a host of other features), which required all command line config. I’m ok with that, although it can be a bit painful. But what’s _really_ bad here is that there’s a convoluted configuration-persistence process you have to go through, otherwise, making changes in the UI _wipes out all other configuration._ Read that again. This is an overwrite, not a merge. Pretty unbelievable.

I spent the other night getting the tunnel up and running. This wasn’t that bad, really, just what you’d expect. Phase1/Phase2 config, some NAT config, etc. Tunnel connected — all was good outbound (house → Azure), but not inbound. Spent a couple of days fighting it and finally it dawned on me I was missing ACLs. Anyway — here’s the CLI config for the S2S tunnel:

set vpn ipsec esp-group azure-esp  
set vpn ipsec esp-group azure-esp lifetime 3600  
set vpn ipsec esp-group azure-esp pfs disable  
set vpn ipsec esp-group azure-esp mode tunnel  
set vpn ipsec esp-group azure-esp proposal 1  
set vpn ipsec esp-group azure-esp proposal 1 encryption aes256  
set vpn ipsec esp-group azure-esp proposal 1 hash sha1  
set vpn ipsec esp-group azure-esp compression disable

set vpn ipsec ike-group azure-ike  
set vpn ipsec ike-group azure-ike lifetime 28800  
set vpn ipsec ike-group azure-ike proposal 1  
set vpn ipsec ike-group azure-ike proposal 1 dh-group 2  
set vpn ipsec ike-group azure-ike proposal 1 encryption aes256  
set vpn ipsec ike-group azure-ike proposal 1 hash sha1

set vpn ipsec ipsec-interfaces interface eth0  
set vpn ipsec logging log-modes all  
set vpn ipsec nat-traversal enable

set vpn ipsec site-to-site peer <Azure Gateway IP>  
set vpn ipsec site-to-site peer <Azure Gateway IP> local-ip <Local Public IP>  
set vpn ipsec site-to-site peer <Azure Gateway IP> authentication mode pre-shared-secret  
set vpn ipsec site-to-site peer <Azure Gateway IP> authentication pre-shared-secret <Azure Key>  
set vpn ipsec site-to-site peer <Azure Gateway IP> connection-type initiate  
set vpn ipsec site-to-site peer <Azure Gateway IP> default-esp-group azure-esp  
set vpn ipsec site-to-site peer <Azure Gateway IP> ike-group azure-ike

set vpn ipsec site-to-site peer <Azure Gateway IP> tunnel 1  
set vpn ipsec site-to-site peer <Azure Gateway IP> tunnel 1 esp-group azure-esp  
set vpn ipsec site-to-site peer <Azure Gateway IP> tunnel 1 local subnet <Local Subnet>  
set vpn ipsec site-to-site peer <Azure Gateway IP> tunnel 1 remote subnet <Azure Subnet>  
set vpn ipsec site-to-site peer <Azure Gateway IP> tunnel 1 allow-nat-networks disable  
set vpn ipsec site-to-site peer <Azure Gateway IP> tunnel 1 allow-public-networks disable

set service nat rule 5000 description 'NAT to Azure'  
set service nat rule 5000 destination address <Azure Subnet>  
set service nat rule 5000 exclude  
set service nat rule 5000 log disable  
set service nat rule 5000 outbound-interface eth0  
set service nat rule 5000 source address <Local Subnet>  
set service nat rule 5000 type masquerade

#these were my existing NAT rules. You should probably inspect your own config before just mindlessly blasting this out  
set service nat rule 5001 description 'MASQ corporate\_network to WAN'  
set service nat rule 5001 log disable  
set service nat rule 5001 outbound-interface eth0  
set service nat rule 5001 protocol all  
set service nat rule 5001 source group network-group corporate\_network  
set service nat rule 5001 type masquerade

set service nat rule 5002 description 'MASQ voip\_network to WAN'  
set service nat rule 5002 log disable  
set service nat rule 5002 outbound-interface eth0  
set service nat rule 5002 protocol all  
set service nat rule 5002 source group network-group voip\_network  
set service nat rule 5002 type masquerade

set service nat rule 5003 description 'MASQ remote\_user\_vpn\_network to WAN'  
set service nat rule 5003 log disable  
set service nat rule 5003 outbound-interface eth0  
set service nat rule 5003 protocol all  
set service nat rule 5003 source group network-group remote\_user\_vpn\_network  
set service nat rule 5003 type masquerade

set service nat rule 5004 description 'MASQ guest\_network to WAN'  
set service nat rule 5004 log disable  
set service nat rule 5004 outbound-interface eth0  
set service nat rule 5004 protocol all  
set service nat rule 5004 source group network-group guest\_network  
set service nat rule 5004 type masquerade

#this is the new rule  
set firewall name WAN\_IN rule 2 action accept  
set firewall name WAN\_IN rule 2 description "azure-networks in"  
set firewall name WAN\_IN rule 2 log disable  
set firewall name WAN\_IN rule 2 protocol all  
set firewall name WAN\_IN rule 2 source group network-group   
\# OR - I'm using a network-group with the Azure subnets, if you're not just use the subnet directly  
set firewall name WAN\_IN rule 2 source address   
delete firewall name WAN\_IN rule 2 state

#this was previously rule 2 - your existing ones may be different so plan accordingly  
set firewall name WAN\_IN rule 3 action drop  
set firewall name WAN\_IN rule 3 description "drop invalid state"  
set firewall name WAN\_IN rule 3 state established disable  
set firewall name WAN\_IN rule 3 state invalid enable  
set firewall name WAN\_IN rule 3 state new disable  
set firewall name WAN\_IN rule 3 state related disable