---
title: "Kinect + Windows 8 + Metro–Part 1: The Backstory"
date: 2012-01-12T05:13:00.000Z
author: "John Patrick Dandison"

---

_This is part one of a series (_[_part two_](http://jpd.ms/post/2012/01/12/Kinect-Windows-8-Metro–Part-2-The-First-Iteration.aspx)_); most of the technical detail will be in the last post. Between NDAs and work disclosure, I can’t release any code, but I can discuss concepts, caveats and successes in the hopes that it helps someone else as much as the open source Kinect projects &amp; the Kinect for Windows community has helped me._

I started writing this and realized it’ll be long. So I’m going to break it up into sections. Section one, the backstory.

My current project at work is pretty sweet. I’m to build an interactive system, accessible and easily usable to guys in gloves — i.e., no touchscreens — to manipulate project data, project images and project plans. Sounds easy enough, huh?

#### Starting out: Touchscreen

When I initially heard about this project, there was a lot of chatter around using a touchscreen. Let’s review why that’s a bad idea:

*   Environment: construction site. I think this says it all:
*   ‘Oh, see, just look right here — oops.’ Drill through screen.
*   Cost: Touchscreens the size we were looking at (42”+) are expensive. I mean _expensive._ Especially with a shortened service life due to environment. Like $3k. $3k + 6–8 month service life? No thanks.
*   User experience: the majority of our users are going to be construction workers. What do construction workers do? They build things. They also wear gloves. Gloves + touchscreen = no worky.

#### Touchscreen Alternatives: Kinect

Kinect popped into my head, what with the dev community doing the cool things they’ve been doing — I expected to be looked at like I was crazy. After the initial questions, business people started throwing around the words ‘Minority Report’ — tip: that’s an exciting time. That’s when business need meets holy-f*n-hell-this-is-sweet dev work — it’s win-win career gold. Champagne falls from the heavens, velvet ropes part — the works.

Here’s what makes the Kinect a great tool for this application:

*   Environment: construction site. I think this says it all:
*   Enclosed plexiglass box. No unsolicited poking. No accidental impalement.
*   Cost: a 42” 1080p TV can be scooped up cheap now. I think the one we found was around $600 for an LG LCD.
*   Kinect Sensor: $150. $99 over the holidays (although with the Kinect for Windows announcement, that looks to be going up to $249, but $100 is cheap for a commercial license — so stop whining about the cost differential)
*   $600 TV + $150 Kinect = $750 — AND the service life expectancy is greater. Fewer replacement cycles at lower cost.
*   User experience: besides the fact that it’s sweet as hell, it’s natural and easy to pick up, from the project exec in the trailer to the guy welding massive steel beams together.

Bonus! Voice.

There’s something else we get for free with Kinect — voice. The Kinect microphone array is killer, so while we haven’t done any field testing, it’s expected to be able to compete at some level with a lapel mic.

#### So that’s the deal. Next up, [the first iteration](http://jpd.ms/post/2012/01/12/Kinect-Windows-8-Metro–Part-2-The-First-Iteration.aspx).
