---
title: "Powershell, Office 365 & Authenticated Proxies"
date: 2013-12-21T00:58:11.000Z
author: "John Patrick Dandison"

---

### Powershell, Office 365 &amp; Authenticated Proxies

#### Short &amp; Sweet

It’s the Friday before Christmas so I’m about to get outta here like the fat kid in dodgeball, but this came across my desk @ WTFHQ today. Using CSOM from behind an authenticated proxy? With events added in PS 2.0, it’s easy.

#### C#.
`var securePassword = new SecureString();  
&#34;sharepoint password&#34;.ToList().ForEach(securePassword.AppendChar);  
var ctx = new ClientContext(&#34;https://sharepoint site&#34;) { Credentials = new SharePointOnlineCredentials(&#34;sharepoint UPN&#34;, securePassword) };  
ctx.ExecutingWebRequest += (s, e) =&gt;  
{  
	e.WebRequestExecutor.WebRequest.Proxy.Credentials = new NetworkCredential(&#34;proxy username&#34;, &#34;proxy password&#34;);  
	//OR  
	e.WebRequestExecutor.WebRequest.Proxy.Credentials = System.Net.CredentialCache.DefaultCredentials;  
};  
var web = ctx.Web;  
ctx.Load(web);  
ctx.ExecuteQuery();`

#### Powershell. Not that different.
`Add-Type -Path &#34;C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\15\ISAPI\Microsoft.SharePoint.Client.dll&#34;  
Add-Type -Path &#34;C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\15\ISAPI\Microsoft.SharePoint.Client.Runtime.dll&#34;  
$pwd = Read-Host -AsSecureString  
$user= &#34;sharepoint UPN&#34;  
$ctx = New-Object Microsoft.SharePoint.Client.ClientContext(&#34;https://sharepoint site&#34;)  
$ctx.Credentials = new-object Microsoft.SharePoint.Client.SharePointOnlineCredentials($user, $pwd)``Register-ObjectEvent -InputObject $ctx -EventName ExecutingWebRequest -Action {   
	$request = $EventArgs.WebRequestExecutor.WebRequest  
	Write-Host &#34;Adding proxy to WebRequest, hold plz&#34;  
	$request.Proxy.Credentials = new System.Net.NetworkCredential(&#34;proxy username&#34;, &#34;a password&#34;)  
	#or, to use default credentials  
	$request.Proxy.Credentials = New-Object System.Net.CredentialCache.DefaultCredentials  
}``$ctx.Load($ctx.Web)  
$ctx.ExecuteQuery()  
Write-Host $ctx.web.Title`
