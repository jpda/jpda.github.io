---
title: "Availability vs. Consistency, through the eyes of a toddler."
date: 2015-06-16T03:23:02.000Z
author: "John Patrick Dandison"

---

![Fred.](http://jpd.ms/wp-content/uploads/2015/06/TY-Giraffe-Tiptop-aus-Pluesch-sowie-Stoff.jpg)

Fred.



_More cloud patterns in real life — this one, while a little silly, illustrates availability, geographic redundancy, rolling upgrades and consistency. Enjoy._

#### Meet Fred.

This is Fred. Fred is my son’s number one — confidant, partner, companion and sleeping buddy. The Hobbes to his Calvin, if you will.

Fred was there when Patrick was born, goes with him nearly everywhere and is as much a part of Patrick’s daily life as his parents. He travels with Patrick, be it to the park or 800 miles away to see his cousins: where Patrick goes, Fred follows. He’s a requirement for sleep — in fact, if presented with Fred, Patrick’s thumb immediately meets his face, regardless of the time of day or current activity. Fred can’t be in sight during bath time or dinner or else Fred too will get a bath or a face full of beans, usually against Patrick’s mother’s wishes.

But we’ve had some availability challenges with Fred — notably, forgetting him when we travel. At least twice we’ve had to order a Fred for a trip, after having realized at the airport that Fred was still in the car or left at the back door. Ordering is a snap — 90 seconds on Amazon and a fresh Fred is in a box and on the way, usually arriving before we do. This has left us with multiple Freds (I think we’re up to 3 now).

#### 99.95% SLA for Fred availability.

Ever had to console an upset baby at 3am? It’s not much fun. The proximity of any available Fred is important, especially in the middle of the night.

Three Freds floating around means I can have a much higher SLA on finding Fred for bed than when we only had one. Fred’s currently in scheduled maintenance in the washing machine? No problem, we’ve got another. Dropped him in the park? Covered in mud? In Dad’s car? All of these issues melt away.

You could say that we’re at high availability with Fred now — with three different Freds, the chances of one being within reach go up dramatically, especially when there’s a primary with known replicas. Since the beginning of 2015, I think we’re at around 99.95% availability of Fred — just about 5 minutes per week on average of unavailability. With the addition of two extra Freds, that number will probably close out even higher by 2016.

Your data is similar. Replicas of your data offer you significantly better availability than a single copy of the data. Imagine if you had to reboot the machine for scheduled maintenance or had a disk failure. In these cases, your data becomes unavailable. Ok if you’re a 20 month old, not ok if it’s business critical data in the middle of the day. Multiple copies means you can get service regardless of the underlying state — if a disk has failed in your RAID array, or

#### Fred as a Service

Fred has transcended being just another stuffed toy. He is the embodiment of security to our little one, just like Linus’ security blanket or whatever your toy of choice was when you were a baby. Because of this, it’s less about a single, distinct Fred and more about the _idea_ of Fred. The physical Fred is really just a host of the Fred idea.

Because we’ve got multiple Freds at our disposal, this makes things immediately easier. We can roll in Fred B while Fred A is getting some stitches upgraded or some emergency maintenance because of a too-available ketchup bottle. Once Fred has been repaired and the pieces of hot dog pulled out of his stuffing, he can return to regular service rotation, all with no loss of service. Do this over and over again, and eventually they’re all up-to-date while being available the whole time.

This is all well and good locally, but doesn’t always work — especially when traveling. Fred (or, more accurately, Patrick’s parents) has had some challenges during travel. He’s been left behind — in the car, at home, etc at least twice during our last few trips. Fortunately, we can order another Fred and have him shipped in minutes. Because of this, there are Freds scattered about NC and eastern Michigan. This gives us a little geographic redundancy — if our house is unavailable, because we’re traveling, we’ve got another Fred available at our destination. This sort of geographic availability works as an additional layer of protection on top of our existing local availability. In fact, if we wanted to get really fancy, we could order multiple Freds for each distinct location. This isn’t always necessary, however — and your application may encounter the same question.

At some point, ultra-redundancy and availability leave cost efficiency. Is it really a requirement that geographic locations where we may spend 2–4 weeks per year have a full-blown Fred deployment? Triple redundancy at all locations? Perhaps we may experience some downtime during those windows, but overall, our SLA isn’t damaged too badly. Not to mention we can get new instances of Fred on-demand within about 12–18 hours. When you’re designing your application’s redundancy and availability requirements, this is something to keep in mind. Cloud brings us a lot of of cost efficiency, but like anything else, it can be abused and cost a fortune. Balancing availability vs. cost is a delicate feat that really only you, as the application owner or data owner can decide.

#### Consistency

FaaS (Fred as a Service) has worked well for us over the last 12 months or so, however, as children grow older, *one* always becomes the ‘primary.’ The rest are just imposters. Similar, yes, but not *the* original Fred. OG Fred has some unique traits — he smells a little different, his fur is a little fuzzier, his neck a little more limp from all the stuffing being pushed elsewhere, his colors a little less vibrant — but he’s familiar. He’s the original.

Because of this, we now have a new problem — OG Fred != Fred B or Fred C. In fact, Fred B and Fred C have been used so infrequently compared to OG Fred that they are in much better (and thus, much different) shape. This presents two problems:

*   Our Freds are now wildly inconsistent — operations to OG Fred haven’t been effectively replicated to Freds B and C.
*   The longer we ignore the problem, the more inconsistent they become.

How do we combat this? We need some way to keep our Freds consistent so that *any* Fred can be OG Fred. There are a couple of ways to combat this. At a minimum, all Freds need to be in regular rotation. It might hurt a bit at first, but it’s about the only way. Or perhaps we speed up the process with the effort of a one-time migration and wear them in a bit — ‘routine maintenance’ with an extra few spin cycles, some loss of stuffing, etc. followed by a reintroduction back into a more stringent rotation.

But a lot of this depends on the client; in this case, a near two year old who doesn’t understand that ‘replication takes time.’ Upset two-year-olds have just recently found their inner divas and as such, will be as picky as humanly possible and find even the most imperceptible of inconsistencies.

But in some cases, you **need** this kind of high consistency. Data consistency comes second to data classification. What kind of data needs high consistency? Credit card/payment data comes to mind immediately — if I make an operation to _take some money_ I had better damn well make sure that succeeds before anything else happens — customers with duplicate charges don’t stay customers for long.

But not all data needs that — if I don’t see your Facebook post the absolute instant that you’ve written it, that’s OK. I’ll see it within a few seconds and there isn’t really any sort of side effect, so it’s totally ok to just keep that data available and not highly consistent. Performance and transaction rate are part of the consideration as well — waiting on multiple geographies to replicate data could take too long when a user is waiting, especially if the data’s consistency requirements don’t dictate that kind of wait. Perhaps wait for 1+ write and async the rest to durable storage.

Other patterns produce similar requirements, with different outcomes — for example, Patrick’s newest trick is to take _all three_ Freds to bed. This presents a problem: FaaS availability is now dependent upon all three being available, not the individual availability of each Fred instance. Now our SLA is almost certainly destined for the tank. Redundancy patterns should consider the level of required consistency when building out both replication and failover strategies. For example, perhaps synchronous writes to 3+ storage providers, while the remainder of writes happen asynchronously for failover only.




![A boy and his Fred](http://jpd.ms/wp-content/uploads/2015/06/WP_20150516_14_03_49_Pro-576x1024.jpg)

A boy and his Fred
