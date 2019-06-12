---
title: 'Shared Workstations, ADFS & SSO (or, just who the *hell* do you think I am?!)'
description: ''
date: '2014-01-14T03:17:25.000Z'
categories: []
keywords: []
slug: >-
  /@jpda/shared-workstations-adfs-sso-or-just-who-the-hell-do-you-think-i-am-c5a753e4137
---

An interesting problem came across my desk at WTFHQ this week. Then it asked me to drop trau and cough.

#### Shared Workstations.

Shared workstations. Used by those in the most chaotic of workplaces, medicine. Nurses & doctors going from patient to patient don’t have time to log out/log in to each machine they use, so in many cases, a ‘guest’ type user is logged on and everyone uses the browser to get stuff done. That’s all well and good, except when you’re talking about SSO & ADFS with Office 365. Whatever user you’re logged into the machine as is who ADFS will authenticate you as, regardless of what you type into the Office 365 login fields. You can see this for yourself — next time you login to O365, type HUGEWATERMELON@yourdomain.com — provided ‘yourdomain.com’ is correct, you’ll be redirected to your ADFS, at which point NTLM takes over and signs you in as whoever you’re signed into the machine as. Anyway, if you’re logged into a guest account, what do you think happens? If that guest account has a mailbox, you’ll go to the mailbox, which is almost certainly \*not\* what you wanted to do.

#### Forms as far as the eye can see.

Forms authentication in ADFS is one way around this problem, but it’s a pretty lame user experience for those on corporate PCs. I’ll hit O365, type in my UPN, get redirected, then type it in \*again\* with my password. Pretty awful, huh? Almost as awful as…well, going to the doctor in the first place.

#### Explanation of Benefits

There’s a solution however. It has some prerequisites on your part, but they shouldn’t be too hard.

Here’s a simple example — I’m going to use reverse DNS to get the IP of the request, see if it’s internal (or in a specific subnet, or whatever), and if the reverse DNS name has my domain name in it, I’ll assume it’s an internal PC.

Extrapolate that further and you could check if that PC is in a specific OU, or whatever. Alternatively, you could see if the user is a specific guest/service account and redirect that way.

Anyway, once you’ve decided what to do with your request, either to let them login with Forms or through NTLM, you need to make sure you redirect and  
_include the original query string._ This is quite important, as it’s just not going to work without.

#### Rx

In ADFS 2.0 and 2.1, all of these files are in IISROOT\\adfs\\ls\\FormsSignIn.aspx.cs. ADFS 3.0 uses HTTP.sys, I haven’t dug through it all yet but I’ll update when I get to it.

Send all requests to FormsSignIn.aspx by modifying the web.config to put Forms first.

<microsoft.identityServer.web>  
    <localAuthenticationTypes>  
        <add name="Forms" page="FormsSignIn.aspx" />  
        <add name="Integrated" page="auth/integrated/" />  
        <add name="TlsClient" page="auth/sslclient/" />  
        <add name="Basic" page="auth/basic/" />  
    </localAuthenticationTypes>  
    ...  
</microsoft.identityServer.web>

When the page loads, you need a way to differentiate requests based on \*some\* amount of information you have from the requesting party. This example used IP & reverse DNS, that’s kinda lame but you could get deep here and check AD for the computer’s OU, check the user for a group, etc.

protected void Page\_Load(object sender, EventArgs e)  
{  
    var ctx = System.Web.HttpContext.Current;  
    //get the query - this is critical, as it contains all the info ADFS needs to process the request  
    var query = ctx.Request.Url.Query.ToString();  
    //do whatever you need to figure out if the machine is shared or not. for this I'm just checking IP  
    var results = System.Net.Dns.GetHostEntry(GetIp(ctx));  
    //doing reverse DNS (since my DNS is authoritative for my house) and checking for a domain name  
    //be able to support that differentiator in your infrastructure - if you filter on OU, for instance, make sure you can get to your AD.  
    if (results.HostName.IndexOf("home.johndandison.com") > -1)  
    {  
        //if it isn't a shared workstation and needs to use integrated authentication, redirect to the NTLM endpoint  
        //\*maintaining the existing querystring\*  
        Response.Redirect("/adfs/ls/auth/integrated/" + query);  
    }  
    //there is no else, since they need to see the forms login page if they are on the shared workstation.  
}

#### The ‘send my kids to college’ method

If you really wanted to do this the right way (i.e., you could apply a service pack and not worry about it getting overwritten), you could write an HTTP handler that does all of your custom logic, which is exactly what Microsoft is doing with their integrated  
handler.

#### \*cough\*