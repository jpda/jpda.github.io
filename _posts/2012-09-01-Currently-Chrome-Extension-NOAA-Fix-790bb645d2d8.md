---
title: Currently Chrome Extension NOAA Fix
description: ''
date: '2012-09-01T17:18:00.000Z'
categories: []
keywords: []
slug: /currently-chrome-extension-noaa-fix-790bb645d2d8
---

**UPDATE — the original creators have updated their extension to use wunderground. I’ll be taking mine down today, you can get their original one below.**

There’s a great Google Chrome extension called [Currently](https://chrome.google.com/webstore/detail/ojhmphdkpgbibohbnpbfiefkgieacjmh) that replaces your ‘new tab’ screen with the current time, current weather & the next four days’ forecasts. I’ve been using it for a few weeks and really enjoyed it’s simplicity, but unfortunately it stopped working the other day…a casualty of the Google Weather API being shut down.

So I decided I’d try to hack something together, at least for my own use. It uses NOAA data (which, unfortunately makes is US only, for now), and it requires a lat/lon vs. only a zip or city name.

I’m releasing it here as an unpacked extension ([download the zip](http://johndandison.com/stuff/bettercurrently.zip), extract it, then go to extensions in Chrome and hit ‘load unpacked extension’), and as a CRX.

Also available in the Chrome Web Store here: [https://chrome.google.com/webstore/detail/fifjopcfeoldcjhigpmpkaoikjaloamm](https://chrome.google.com/webstore/detail/fifjopcfeoldcjhigpmpkaoikjaloamm "https://chrome.google.com/webstore/detail/fifjopcfeoldcjhigpmpkaoikjaloamm")

Caveats:

* Probably US only (may switch to MSN weather api, but for now it’s NOAA data which I’m assuming is going to be US only)
* Requires latitude/longitude from geolocation should be fixed for using zip code/address. still US only.
* the F to C conversion doesn’t seem to be working properly (I suspect 72F != 1)

If I can move to the MSN API & find a good zip-to-lat/lon converter, then I should be able to remedy those issues, although by then the original creators may have updated their extension.
