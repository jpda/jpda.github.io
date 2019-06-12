---
title: 'Powershell, Office 365 & Authenticated Proxies'
description: ''
date: '2013-12-21T00:58:11.000Z'
categories: []
keywords: []
slug: /powershell-office-365-authenticated-proxies-391a7494570a
---

It’s the Friday before Christmas so I’m about to get outta here like the fat kid in dodgeball, but this came across my desk @ WTFHQ today. Using CSOM from behind an authenticated proxy? With events added in PS 2.0, it’s easy.

#### C#

```c#
var securePassword = new SecureString();  
"sharepoint password".ToList().ForEach(securePassword.AppendChar);  
var ctx = new ClientContext("https://sharepoint site") { Credentials = new SharePointOnlineCredentials("sharepoint UPN", securePassword) };  
ctx.ExecutingWebRequest += (s, e) =>  
{  
	e.WebRequestExecutor.WebRequest.Proxy.Credentials = new NetworkCredential("proxy username", "proxy password");  
	//OR  
	e.WebRequestExecutor.WebRequest.Proxy.Credentials = System.Net.CredentialCache.DefaultCredentials;  
};  
var web = ctx.Web;  
ctx.Load(web);  
ctx.ExecuteQuery();
```

#### Powershell. Not that different.

```powershell
Add-Type -Path "C:\\Program Files\\Common Files\\Microsoft Shared\\Web Server Extensions\\15\\ISAPI\\Microsoft.SharePoint.Client.dll"  
Add-Type -Path "C:\\Program Files\\Common Files\\Microsoft Shared\\Web Server Extensions\\15\\ISAPI\\Microsoft.SharePoint.Client.Runtime.dll"  
$pwd = Read-Host -AsSecureString  
$user= "sharepoint UPN"  
$ctx = New-Object Microsoft.SharePoint.Client.ClientContext("https://sharepoint site")  
$ctx.Credentials = new-object Microsoft.SharePoint.Client.SharePointOnlineCredentials($user, $pwd)

Register-ObjectEvent -InputObject $ctx -EventName ExecutingWebRequest -Action {   
	$request = $EventArgs.WebRequestExecutor.WebRequest  
	Write-Host "Adding proxy to WebRequest, hold plz"  
	$request.Proxy.Credentials = new System.Net.NetworkCredential("proxy username", "a password")  
	#or, to use default credentials  
	$request.Proxy.Credentials = New-Object System.Net.CredentialCache.DefaultCredentials  
}

$ctx.Load($ctx.Web)  
$ctx.ExecuteQuery()  
Write-Host $ctx.web.Title
```
