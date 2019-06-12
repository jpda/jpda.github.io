---
title: Adding Existing VHDs to Azure Resource Groups
description: ''
date: '2015-01-19T04:00:21.000Z'
categories: []
keywords: []
slug: /@jpda/adding-existing-vhds-to-azure-resource-groups-eb3bdfab6322
---

Azure Resource Groups. Simultaneously the most exciting and most frustrating part of Azure vNext. While powerful, today they’re quite inflexible — no API is exposed to allow editing info (like names), moving resources from one to another, really any management at all. And the sprawl_— the sprawl!_ It’s awful. So many auto-generated resource groups. With a finite number allowed per account, actually _using them_ appropriately is priority.

I elected to move some of my older deployments into a new virtual network, to gain the internal load balancer and to bring things on par with what’s current. For VMs, this is generally easy — delete the VM, keeping the disks, then recreate the VM in the new network. No big deal. Through this same way, I could also deploy these machines into the same resource group, containing similar resources.

Until I got into the new portal — I can’t, for the life of me, find any way within the new portal to create a new VM from an existing VHD. I dug through the PowerShell cmdlets for a bit, still couldn’t find much — particularly for adding that new VM into an existing resource group.

_Side note: we still can’t upload our own Resource Group Templates? Really?_

#### Clunk

For now, create a throwaway virtual machine in your resource group, with the proper cloud service name (DNS name, in the new portal), networking, storage account, etc. In the old portal (or through PowerShell), create your new VM from the existing VHD as you would normally, making sure to pick the cloud service which was created in your resource group. You can delete the throwaway VM now too. Check back in the portal a bit later and your existing VHD should now be in a new VM, in a resource group of your choosing.