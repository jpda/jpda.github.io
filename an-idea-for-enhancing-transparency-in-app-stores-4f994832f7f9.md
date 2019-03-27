---
title: "An idea for enhancing transparency in app stores"
date: 2018-09-08T03:17:34.105Z
author: "John Patrick Dandison"

---

![image](https://cdn-images-1.medium.com/max/1200/1*M8dXB74OyChr7EWmGwmgJQ.jpeg)

[Original source](https://www.flickr.com/photos/gabeandchry/9318305000), [CC-BY 2.0](https://creativecommons.org/licenses/by/2.0/)

I saw an article today about apps that send location data to third parties. This isn’t especially new, but it got me thinking. [https://techcrunch.com/2018/09/07/a-dozen-popular-iphone-apps-caught-quietly-sending-user-locations-to-monetization-firms/](https://techcrunch.com/2018/09/07/a-dozen-popular-iphone-apps-caught-quietly-sending-user-locations-to-monetization-firms/)

I’ll preface this to say I’m not really a mobile developer, I have hacked some stuff together in the past, but mostly in support of other things. I also don’t own an app store, so I’m unable to implement this on any sort of broad scale. But I’m thinking of some sort of app-sandbox-whitelisting, that only allows communication to specific endpoints without user consent, and requires user consent for additional, ‘value-added’ services.

I think it would be fairly straightforward, and could be done in a way similar to permission escalations today. Or similar to a browser extension (‘this app requires access to all sites, *.somesite.com, etc):

*   Developer publishes application in store
*   Developer lists endpoints for outbound network communication, marking them as required/essential for functionality or optional/enhanced functionality
*   For required endpoints, developers would need to certify or validate ownership
*   Required endpoints would be approved via installing the app, no escalation required. Perhaps even shown in the app store listing
*   Optional endpoints would require explicit user consent, similar to accessing photos or location or whatever
*   Perhaps some integration with large analytics platforms, like HockeyApp, Firebase, etc, for simplifying app analytics and consent
*   Being able to dynamically update endpoints and get just-in-time access consent

I don’t think this is necessarily intended to _block_ traffic, at least not at the beginning, but about giving users enough information to make an informed decision. Just an idea. Tell me what you think!
