---
title: Updating ADFS 3 for WIA on Windows 10
description: ''
date: '2015-07-30T15:49:33.000Z'
categories: []
keywords: []
slug: /updating-adfs-3-for-wia-on-windows-10-ea8595bdaba5
---

**Updated 7/30/15**

Here’s the latest that’s working with IE 11 on Windows 10 RTM/10240:

Set-AdfsProperties -WIASupportedUserAgents @("MSIE 6.0", "MSIE 7.0; Windows NT", "MSIE 8.0", "MSIE 9.0", "MSIE 10.0; Windows NT 6", "Windows NT 6.4; Trident/7.0", "Windows NT 6.4; Win64; x64; Trident/7.0", "Windows NT 6.4; WOW64; Trident/7.0", "Windows NT 6.3; Trident/7.0", "Windows NT 6.3; Win64; x64; Trident/7.0", "Windows NT 6.3; WOW64; Trident/7.0", "Windows NT 6.2; Trident/7.0", "Windows NT 6.2; Win64; x64; Trident/7.0", "Windows NT 6.2; WOW64; Trident/7.0", "Windows NT 6.1; Trident/7.0", "Windows NT 6.1; Win64; x64; Trident/7.0", "Windows NT 6.1; WOW64; Trident/7.0", "MSIPC", "Windows Rights Management Client", "Windows NT 10.0; WOW64; Trident/7.0; rv:11.0")

If you’re using the Windows Technical Preview, you may notice that ADFS presents you with a Forms login instead of using WIA from IE on a domain machine. This little chunk of powershell includes most of the major browsers that support WIA — you can plunk this into your ADFS server and get it going:

```powershell
Set-AdfsProperties -WIASupportedUserAgents @("MSIE 6.0", "MSIE 7.0; Windows NT", "MSIE 8.0", "MSIE 9.0", "MSIE 10.0; Windows NT 6", "Windows NT 6.4; Trident/7.0", "Windows NT 6.4; Win64; x64; Trident/7.0", "Windows NT 6.4; WOW64; Trident/7.0", "Windows NT 6.3; Trident/7.0", "Windows NT 6.3; Win64; x64; Trident/7.0", "Windows NT 6.3; WOW64; Trident/7.0", "Windows NT 6.2; Trident/7.0", "Windows NT 6.2; Win64; x64; Trident/7.0", "Windows NT 6.2; WOW64; Trident/7.0", "Windows NT 6.1; Trident/7.0", "Windows NT 6.1; Win64; x64; Trident/7.0", "Windows NT 6.1; WOW64; Trident/7.0", "MSIPC", "Windows Rights Management Client")
```

#### Why?

In version 3, ADFS tries to intelligently present a user experience that’s appropriate for the device. Browsers that support WIA (like IE) provide silent sign on, while others (like Chrome, Firefox, mobile browsers, etc) are presented with a much more attractive and user friendly forms-based login. This is all automatically handled now, unlike before where users with non-WIA devices were prompted with an ugly and potentially dangerous basic 401 authentication box (if they were prompted at all).

This means you can now design a login page for non WIA devices that might include your logo, some disclaimers or legal text.

![iOS 8 + ADFS 3IMG_0001](/img/0_sOYJ2ZLs3Xi7GcdU.png)
iOS 8 + ADFS 3
