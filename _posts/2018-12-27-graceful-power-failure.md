---
layout: post
title: "Emergency Exit: Graceful VM (and ESXi!) Shutdown"
subtitle: "Power your services down like going to bed."
date: 2018-12-27 15:45:00
background: '/img/postheads/04.jpg'
---

# What's this?

Something I learned in doing my beginnings with virtualization, I came to realize something: power failures happen WAY more than I think they do. At least in my area. So, with only about a handful of VMs and a host that probably was taking the brunt of power failures more than it should, I figured I had to do something.

At this point, however, I very regularly use APC UPSes, and have used both the PowerChute application and apcupsd. The support for them is pretty good, however, I will fully admit that I've largely been buying consumer UPSses because that's simply what I can afford at this time.

After having had plenty of power failures despite the university pledging that power is always on, (It is, just for the critical infrastructure devices, of which everyone's definition differs) I kept experiencing blackouts and the occasional brownout, even more so when I started living in my own apartment, although that is not the university's fault. I started to search the web for solutions, but they all involved things like APC SmartCards, which I **certainly** did not have the infrastructure for.

However, after having played around with (corrupted) VMs long enough, I learned there was actually a way to make this happen. ESXi has a [NUT client ported for it](https://gist.github.com/InQuize/42fdf629fc77ef5c2d57) that works on versions 5.0, 6.0, and 6.5. So if I couldn't run the NUT server on ESXi, I realized that I could still connect a UPS that would normally connect via PowerChute or apcupsd to the ESXi host and pass it through to a VM that **COULD** run the NUT server.

And then I realized: Not only can that VM run the NUT server on the passed-through USB connection, it could be the NUT server for **ALL VMs ON THE NETWORK!** At this point, it would have been stupid for me to not try it, so I thought "what the hell, let's give it the old college try."

The best part about this thing is that it actually worked pretty well. However, I had some goals that I wanted to achieve:

	* The VMs must start immediately going down on power failure. Clean shutdowns everywhere.
	* The ESXi host MUST go down last, because it will, in a great twist of irony, bring down the NUT server with it.
	* The host must be completely off once the power ultimately runs out on the UPS.
	* There will be no cancellation of the shutdown timers because the power frequently comes in and out. Better to just leave it off.