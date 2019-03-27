---
title: "Media Center Fails DCA due to HDCP Failure"
date: 2013-05-20T23:02:27.000Z
author: "John Patrick Dandison"

---

My desktop at home has an ATI Radeon 6770. I recently moved into a new office at home &amp; decided TV would be best, since it’s a bit larger now, so I wanted to setup my desktop to use the HDHomeRun Prime that runs the TV downstairs. This requires running the infamous ‘Digital Cable Advisor’ in Media Center.

I thought, no sweat — 16GB memory, i7, Radeon 6770, no big deal. The PC downstairs is an old laptop with a *decent* mobile Radeon, so surely this would run it properly…surely.

And of course, the DCA failed, stating my display or driver wasn’t HDCP compliant. After about seven driver version installs/removals, I started looking at my video card wondering why on earth this was happening — it’s pretty simple. Hyper-V. It adds some extra display drivers that MCE doesn’t play well with.

Disable Hyper-V, reboot &amp; DCA will pass. You can re-enable Hyper-V after that &amp; all works in harmony.
