---
title: Network Drives on SBS 2011 Remote Web Folders
description: ''
date: '2011-04-16T06:31:00.000Z'
categories: []
keywords: []
slug: /network-drives-on-sbs-2011-remote-web-folders-b01f5ef1e9f6
---

I installed Small Business Server last weekend here at the house in an attempt to consolidate some boxes and clear some clutter in preparation for the impending move. A really sweet part of SBS is the Remote Web Workspace, which gives you links to all of the internal resources SBS publishes — you get links to OWA, SharePoint, Shared Folders & Remote Desktops. Plus, it’s all extensible, so you can write your own providers to publish whatever data and widgets you want up to your users.

Since I don’t really have ‘users’ per se, just me and my wife, I don’t really care about UI customization. It looks fresh and modern, like the Windows Live redesign. Functionally, I love it — it takes all available resources here at my house and packages them up in an easy to use interface.

One part you’ve got are the shared folders. The interface is spartan, but usable — you get a tree view & a search, and you can download whole folders with two-clicks — it will auto-zip them and fire them over the wire.

Great, you say — until you realize that it’s only good for shares _on the local machine_ (i.e., the Small Business Server box). I have two servers — one running SBS, the other running Server 2008 R2, with Hyper-V and lots of cheap storage. Since almost all of my storage is on the other box, this presented a problem. The SBS Console refuses to add anything but locally-attached drives. I tried all kinds of stuff, from Linux-esque symbolic links to NTFS junctions, none of which worked.

![image](/img/0_-QjxKwJt9xOMfcHV.png)

The more I thought about it, though, the more I realized there _had_ to be a way to get the shares from my other server exposed this way as well…I mean, why have the server as the top-level of the tree view if it’s always the only root?

There is no documentation on this, so I got out reflector. I could go on for an entire post about how RedGate is an awful company for charging now for reflector, but instead I’ll just say this — go get ILSpy.

Before you get into any of this, remember, if you are in a production environment, _please_ test much more thoroughly than I have. I’m only using this with <5 users and it is certainly not a ‘production’ environment. If you’re doing real work, by all means, be careful. This is horribly unsupported and chances are high it would break if service packs or anything broke the underlying implementation. There’s a reason most of these classes are private and internal.

#### Method A: Keeping it Simple

Find the web.config — it is here: Program Files\\Windows Small Business Server\\Bin\\WebApp\\RemoteAccess

Further down, you’ll find the storage provider (among a host of other providers):

<wssg.storageProvider type="Microsoft.WindowsServerSolutions.Web.Storage.SBSStorageProvider,

Wssg.Web.StorageProvider" />

Which is the standard implementation in SBS 2011. Some investigating yielded that it inherited from IStorageInformationProvider.

Some more digging showed that there were some other implementations of IStorageInformationProvider, one of which used a configuration key called ‘shares.’

That one would be the FileSystemBasedStorageInformationProvider, (in Web.Internal) which pulled a list of shares from semicolon delimited list in the config file, from a configuration key called ‘shares.’ It does this in the Initialization method — choosing to use the configuration key if it is not null. This sounds well and good, BUT it is an exclusive or. You’ll get config entries OR file shares…meaning the standard pull from WMI won’t work any longer. With this method, you’ll have to explicitly define each and every share on any machine you want to appear in RWA.

Switching to the FileSystemBasedStorageInformationProvider is simple enough — remove (or comment out) the original line, and add this one right beneath it:

<wssg.storageProvider type="Microsoft.WindowsServerSolutions.Web.Storage.FileSystemBasedStorageInformationProvider,

Wssg.Web.Internal" shares="\\\\servername\\sharename;\\\\servername2\\sharename;\\\\servername3\\sharename" />

Editing the web.config will fire off a re-JIT, so you’ll have to re-sign in. If you did it correctly, you should now see your shares — but _only_ the ones you _explicitly_ added to the config file.

#### Method B: Combining the Best of Both — Building a new StorageProvider

Since I had reflector out and I was already digging through Internal classes anyway, I decided to try and combine the output from both the configuration class and the SBSStorageProvider.

Needless to say, collecting dependencies is always the worst part. First, install the Windows Server SDK (you can get that [here](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=105694e5-76bc-4820-b42c-6f4250b4f5be)). That will give you some of them. The rest you’ll have to grab out of the Program Files\\Windows Small Business Server\\Bin\\WebApp\\RemoteAccess\\Bin folder. Here are the ones you’ll need:

*   Wssg.Web (SDK)
*   Wssg.Web.Internal (from the bin)
*   Wssg.Web.StorageProvider (from the bin)
*   StorageCommon (from the Small Business Server bin — Program Files\\Windows Small Business Server\\Bin)

Unfortunately, the constructors for these are internal. There is probably a very good reason for this, but whatever — this meant I had to basically copy\\paste the disassembled code into my project to get constructors (gross). It is gross, and should have been yet another red flag that this was a bad idea. It was not.

Getting the config file reader to work was pretty much a cake walk. It is a simple implementation and didn’t take much to get into my project. The next one, however, was a bit more of a pain. The basic architecture is this: IStorageInformationProvider returns a generic List<IShareInfo>, which contains all of the information for the shares. The IShareInfo interface is pretty simple too. The concrete class StorageAPIBasedShareInfo, however, is not. It has dependencies all the way into the StorageCommon assembly, which is why it is referenced. Anyway, what these things do is pretty simple — get some shares, check some permissions, and, ultimately, return a generic List<IShareInfo>, which then gets returned out to whatever requesting consumer.

I created my assembly, which you can download below. It basically calls the shamelessly copy\\pasted\\tweaked methods from the two StorageInformationProviders and combines them before sending them back out. All of the caching and permissions mechanisms are still in tow, but again — please test before using this in any kind of production environment. I doubt it’s terribly kosher with the SBS peeps either.

![image](/img/0_D8epqnvGGcKkZ5X8.png)

#### Method B-2: Installing My Provider

Download this assembly: [http://jpd.ms/stuff/sbs/johndandison.SBS.StorageProvider.dll](http://jpd.ms/stuff/sbs/johndandison.SBS.StorageProvider.dll)

Or get the C# source here: [http://jpd.ms/stuff/sbs/SBSAndConfigBasedStorageInformationProvider.cs.txt](http://jpd.ms/stuff/sbs/SBSAndConfigBasedStorageInformationProvider.cs.txt)

Start in this folder on your SBS machine: Program Files\\Windows Small Business Server\\Bin\\WebApp\\RemoteAccess\\

Copy the assembly to the bin folder.

Drop this line into your web.config:

<wssg.storageProvider type="johndandison.SBSAndConfigBasedStorageInformationProvider, johndandison.WSS.StorageProvider"

shares="\\\\server\\share;\\\\server\\share" />

That should be about it. Your box will need to re-JIT, but it shouldn’t take but a few seconds.

#### Addendum: Search

Search doesn’t work. When I have some time to rip the search assembly apart, I’ll attempt to write a search provider that uses all the shares, not just the standard SBS ones. Right now, it won’t search beyond the shares on your SBS machine.