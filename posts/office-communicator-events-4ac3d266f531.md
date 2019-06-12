---
title: Office Communicator Events
description: ''
date: '2011-04-12T05:04:04.000Z'
categories: []
keywords: []
slug: /@jpda/office-communicator-events-4ac3d266f531
---

I needed a breather Friday, so I started playing with the Office Communicator Automation API. Basically, you can sign in, get contacts, modify contact lists, etc. It is an automation API, so the client needs to be installed (or so I saw).

Instantiation and usage is really easy too:

public void SignIn()  
{  
    if (connected) return;  
    if (communicator == null)  
    {  
        communicator = new CommunicatorAPI.Messenger();  
        communicator.OnSignin +=   
            new DMessengerEvents\_OnSigninEventHandler  
                (communicator\_OnSignin);  
        communicator.OnMyStatusChange +=   
            new DMessengerEvents\_OnMyStatusChangeEventHandler  
                (communicator\_OnMyStatusChange);  
    }  
    communicator.AutoSignin();  
}

Basically, I want to do some stuff when I change my status. I’d really like to set my softphone up for forwarding, but that’s a different story for a different day. Wiring up the event is easy enough:

void communicator\_OnMyStatusChange(int hr, MISTATUS mMyStatus)  
{  
    //do some stuff  
}

Anyway, there’s a pretty glaring bug — events don’t fire whenever you reach ‘Inactive’ status.

For instance, I have my settings to set to Inactive after five minutes — but as soon as that happens, my events quit firing. Nothing in the documentation, nothing in the event logs, they just stop firing. I’ve been able to replicate this over and over.

I tried quite a few things — setting the auto-away & auto-inactives to 0 (didn’t work), setting them to ridiculously large numbers (still didn’t work — it looks like the max upper bounds is 60 minutes), still nothing.

Ultimately, I ended up using a little utility that fakes out mouse movement every seconds (one of those ‘never go to sleep’ type apps)…and that seemed to finally work. Only time will tell.

So — the tl;dr version: Communicator events don’t fire when you’re in the ‘Inactive’ state.