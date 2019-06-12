---
title: Removing the SalesLogix ActiveMail ActiveX Control
description: ''
date: '2010-02-10T13:30:26.000Z'
categories: []
keywords: []
slug: /@jpda/removing-the-saleslogix-activemail-activex-control-4cf35d0d791
---

Made the mistake of installing the SalesLogix ActiveMail ActiveX Control? Kicking yourself because there’s no entry in Add\\Remove Programs? Screwed up your Outlook with recklessly coded add-ins? Fret not, there is a way.

Copy the text below into a text file, and save it as <filename>.bat. Run an elevated command prompt and execute the script. This will de-register all of the SalesLogix ActiveMail assemblies and COM servers.

Be sure to note — if you’re using an x64 OS, change the path from c:\\windows\\system32 to c:\\windows\\syswow64.

Enjoy.

rem change path to c:\\windows\\syswow64 for x64 systems

cd c:\\windows\\system32

regsvr32 /u slmn.dll

regsvr32 /u SlxEmailNotifier.dll

regsvr32 /u SLXMMGUIW.dll

regsvr32 /u SLXDocW.dll

regsvr32 /u SLXMMEngineW.dll

regsvr32 /u SLXFaxW.dll

regsvr32 /u SLXWinFaxW.dll

regsvr32 /u SLXFramer.ocx