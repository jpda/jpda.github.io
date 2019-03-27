---
title: "Updating ADFS 3 for WIA on Windows 10"
date: 2015-07-30T15:49:33.000Z
author: "John Patrick Dandison"

---

****Updated 7/30/15****

Here’s the latest that’s working with IE 11 on Windows 10 RTM/10240:
`Set-AdfsProperties -WIASupportedUserAgents @(&#34;MSIE 6.0&#34;, &#34;MSIE 7.0; Windows NT&#34;, &#34;MSIE 8.0&#34;, &#34;MSIE 9.0&#34;, &#34;MSIE 10.0; Windows NT 6&#34;, &#34;Windows NT 6.4; Trident/7.0&#34;, &#34;Windows NT 6.4; Win64; x64; Trident/7.0&#34;, &#34;Windows NT 6.4; WOW64; Trident/7.0&#34;, &#34;Windows NT 6.3; Trident/7.0&#34;, &#34;Windows NT 6.3; Win64; x64; Trident/7.0&#34;, &#34;Windows NT 6.3; WOW64; Trident/7.0&#34;, &#34;Windows NT 6.2; Trident/7.0&#34;, &#34;Windows NT 6.2; Win64; x64; Trident/7.0&#34;, &#34;Windows NT 6.2; WOW64; Trident/7.0&#34;, &#34;Windows NT 6.1; Trident/7.0&#34;, &#34;Windows NT 6.1; Win64; x64; Trident/7.0&#34;, &#34;Windows NT 6.1; WOW64; Trident/7.0&#34;, &#34;MSIPC&#34;, &#34;Windows Rights Management Client&#34;, &#34;Windows NT 10.0; WOW64; Trident/7.0; rv:11.0&#34;)`

If you’re using the Windows Technical Preview, you may notice that ADFS presents you with a Forms login instead of using WIA from IE on a domain machine. This little chunk of powershell includes most of the major browsers that support WIA — you can plunk this into your ADFS server and get it going:
`Set-AdfsProperties -WIASupportedUserAgents @(&#34;MSIE 6.0&#34;, &#34;MSIE 7.0; Windows NT&#34;, &#34;MSIE 8.0&#34;, &#34;MSIE 9.0&#34;, &#34;MSIE 10.0; Windows NT 6&#34;, &#34;Windows NT 6.4; Trident/7.0&#34;, &#34;Windows NT 6.4; Win64; x64; Trident/7.0&#34;, &#34;Windows NT 6.4; WOW64; Trident/7.0&#34;, &#34;Windows NT 6.3; Trident/7.0&#34;, &#34;Windows NT 6.3; Win64; x64; Trident/7.0&#34;, &#34;Windows NT 6.3; WOW64; Trident/7.0&#34;, &#34;Windows NT 6.2; Trident/7.0&#34;, &#34;Windows NT 6.2; Win64; x64; Trident/7.0&#34;, &#34;Windows NT 6.2; WOW64; Trident/7.0&#34;, &#34;Windows NT 6.1; Trident/7.0&#34;, &#34;Windows NT 6.1; Win64; x64; Trident/7.0&#34;, &#34;Windows NT 6.1; WOW64; Trident/7.0&#34;, &#34;MSIPC&#34;, &#34;Windows Rights Management Client&#34;)`

#### Why?

In version 3, ADFS tries to intelligently present a user experience that’s appropriate for the device. Browsers that support WIA (like IE) provide silent sign on, while others (like Chrome, Firefox, mobile browsers, etc) are presented with a much more attractive and user friendly forms-based login. This is all automatically handled now, unlike before where users with non-WIA devices were prompted with an ugly and potentially dangerous basic 401 authentication box (if they were prompted at all).

This means you can now design a login page for non WIA devices that might include your logo, some disclaimers or legal text.




![IMG_0001](http://jpd.ms/wp-content/uploads/2014/12/IMG_0001-1024x768.png)

iOS 8 + ADFS 3
