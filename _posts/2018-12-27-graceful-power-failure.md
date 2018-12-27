---
layout: post
title: "Emergency Exit: Graceful VM (and ESXi!) Shutdown"
subtitle: "Power your services down like going to bed."
date: 2018-12-27 15:45:00
background: '/img/postheads/04.jpg'
---

# What's this?

Something I learned in doing my beginnings with virtualization, I came to realize something: power failures happen WAY more than I think they do. At least in my area. So, with only about a handful of VMs and a host that probably was taking the brunt of power failures more than it should, I figured I had to do something.

At this point, however, I very regularly use APC UPSes, and have used both the PowerChute application and apcupsd. 