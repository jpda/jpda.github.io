---
title: 'Kinect + Windows 8 + Metro–Part 2: The First Iteration'
description: ''
date: '2012-01-13T04:06:00.000Z'
categories: []
keywords: []
slug: /kinect-windows-8-metro-part-2-the-first-iteration-9901ec74c2bd
---

_This is part two of a series (_[_part one_](http://jpd.ms/post/2012/01/12/Kinect-Windows-8-Metro%E2%80%93Part-1-The-Backstory.aspx)_); most of the technical detail will be in the last post. Between NDAs and work disclosure, I can’t release any code, but I can discuss concepts, caveats and successes in the hopes that it helps someone else as much as the open source Kinect projects & the Kinect for Windows community as a whole has helped me._

I had gotten the green light, so now this was exploratory. I decided the best place to start would be with some of the toolkits and samples that the community had created. I grabbed [KinectNUI](http://kinectnui.codeplex.com) & the [Kinect Toolbox](http://kinecttoolbox.codeplex.com/). KinectNUI has a lot of legwork done in gesture detection, smoothing & interop with traditional devices (mouse/keyboard). The Kinect Toolbox has a really sweet templated gesture detection engine, as well as a playback/record system that lets you debug without jumping out of your seat (ah, laziness IS the mother of invention).

#### Windows 7 + WPF + Kinect Public SDK

I started out extending the KinectNUI — my experience with WPF has been limited, so it was a good opportunity to sharpen that skillset. I wiped most of the KinectNUI out, leaving me with the old ‘MainWindow,’ which was now a ‘diagnostics’ window. In fact, it was now a user control, as well as all of my other content pages — so I could get some animations between controls to give a more ‘Xbox Dashboard’ effect.

#### MainWindow

It started simple enough — I made my XAML window the same size as the screens our prototype would be running on — 1080p HDTVs. The camera was pumped to a live-view canvas in the bottom corner (a la Dance Central), our logo was in the top left corner, and our main content viewport was a canvas in the middle with some animations for swapping user controls. The ‘MainWindow’ root canvas had the Kinect hands painted on top of that whenever a Skeleton frame was ready.

#### Tiles/Canvas

So our main content viewport is the good stuff — all the content goes here. Tiles with actions across the screen; hovering on a tile (or clicking) executes the action associated with that tile. Quite simple really — I wrote a base class — we’ll call it ‘ASweetKinectTile’ — which inherits from UIElement. My tiles inherit from ASweetKinectTile, with some override-able methods (say, SwipeLeft, SwipeRight, Click, etc), making virtually any element Kinect-enabled.

#### Hover-to-Activate

This proved interesting — and simply came down to mapping the coordinates of the ‘tiles’ on the screen & counting skeleton frames while a specific joint (or combination of joints) was inside the bounds of that element. I’m sure some WPF wizard could make it happen in some ‘proper’ way, but this was rapid prototyping — working with temporary hax is a lot better than ‘not working, but no hax’ — anyway, it worked quite swimmingly for a while.

#### Gestures

While the gesture detection in the KinectNUI worked fairly well, I read some good things about the TemplatedGestureDetector in the Kinect Toolbox, so I decided to give it a shot. I will say this now and probably later. Gesture detection is a pain. Not the detecting gestures part, but the ignoring inadvertent movement part. I’ve been noodling a couple of ways to avoid this (particularly with left/right swipes — the tendency is to swipe over, then move your hand back to swipe again, essentially swiping back in the other direction) — namely, a delay between detections.

#### Some Thoughts

While the solution was good, we liked it, the business people liked it, the UI was still far from polished — it was a prototype, after all (and only the first sprint). So I got to thinking: Tiles = metro. Full screen = metro. How about porting over the prototype into Windows 8? It seemed a natural fit. Next post — Windows 8 & a Metro app.