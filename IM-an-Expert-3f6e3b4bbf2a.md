---
title: "IM an Expert"
date: 2011-06-29T02:31:00.000Z
author: "John Patrick Dandison"

---

A little blip came up on the Microsoft radar a couple of weeks ago; an application (well, really a few applications) all bundled together called ‘IM an Expert.’ The whole premise is ‘people helping people’ via intelligent, context-sensitive matchmaking.

Here is the basic workflow:

*   Guy says ‘hey, I need help with X.’
*   Magic happens, finding expert to match with query. Lots of unicorn dust in here (after review, it’s a lot less magical — virtually no dust at all, unless you consider SQL full-text indexing to be ‘magical.’ I do not.)
*   Expert says, ‘I am an X expert, I’ll be happy to help.’
*   Conversation is recorded and people can check it out on a website.
*   People can also register to be experts on this site.

Pretty simple, eh? There is an indexer &amp; web crawler as well, which will go out to the wild wild west of the internet and crawl content you’ve marked as relevant to you, as well as your SharePoint My Site.

It sounds really cool — and since enterprise IM is a big deal these days, it is a good marketing tool as well.

I got it all setup (which involved 2x DCs for a new test domain, a Lync 2010 server, a server to run the IManExpert Bot &amp; website, &amp; two clients.

#### You’re all installed, followed the instructions and…it’s broken?!

Two of the appSettings keys are incorrect. The BotPassword &amp; BotUsername keys are missing, with the EmailUsername &amp; EmailPassword keys being duplicated.

#### Now we’re running…so what else can we do with this?

I wanted to break free from Lync a bit, maybe try a web interface, so I wrote a quick web-chat client using the IManExpert website-provided API ([http://imanexpert/api.aspx](http://imanexpert/api.aspx)).

This was, well, less than successful. It worked great up until the point where I logged off of one of the machines. Then the conversation basically went all to hell.

I was annoyed and started disassembling it, thinking surely Microsoft wrote this in a reusable manner.

#### This is not the case.

In short, here’s what is mad lame about it –

#### it (appears) it was released as a way to move Lync licenses

not to be actually truly functional. Since it is so tightly coupled to Lync, creating a conversation using the API actually opens Lync conversations, and requires experts\participants to type in those. You can submit some text into the conversation via the API, but ultimately, if you don’t have Lync open, it will not work, period. Pretty lame, eh?

#### So now what?! All this fanfare for nothing? Sounds like the SharePoint 2007 BDC!

Yes, it certainly does. Only the BDC is something people _actually paid for_ that really didn’t do anything. This is free — but fret not, for there is a saving grace:

#### it is entirely in .net.

Yes, get your favorite disassembler and get to work. I’d say break out reflector (oh, wait, you can’t do that, because [redgate](http://red-gate.com) sucks. They took a community tool, promised to keep it free, then started timebombing _and deleting_ people’s old copies! Needless to say, I’ll never buy another redgate product). I have spent a bit getting the decompiled projects running, and I must say, the LyncInterface dependencies are staggering. I’m venturing a guess that what Microsoft has given us is merely a façade, a ‘look at what we did (five years ago),’ not what they really use inside.

#### Now what? And what on earth is this article even about?

That is a good question. I’ve disassembled everything but there isn’t much useful inside of it. I build out a whole test domain to try this out, and was sorely disappointed. I can’t really say much else since I’m working on this at my job, but it will be SWEET when it is finished. The only advice I can give is this: don’t try to reuse the IM an Expert code, database, or anything, really, because it’s pretty tightly coupled to Lync, and that is lame. Maybe if my company doesn’t mind, I’ll share some of what I end up with later down the road. Until then, enjoy buying Lync licenses.
