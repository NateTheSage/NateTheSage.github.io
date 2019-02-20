---
layout: post
title: "Diana, Part One: Home Email, And Then Some"
subtitle: "Easy e-mail, and running your own."
date: 2018-12-22 13:30:00
background: '/img/postheads/03.png'
---

## Why am I writing about this?

I'm sure those of you reading this have a GMail account. Of course, a lot of us do, GMail has over 1.5 billion users as of 2018. They have a nearly one-quarter market share, and GMail is accessed by three-quarters of those users on mobile devices. Of course, you could get all this from Google, they very much like to brag about their service usage.

The problem is that Google, purveyor of great (terrible) sayings like "don't be evil" (you've now given me license to be such), and just assuming that you say yes to their collection practices, has gotten to the point where GMail is no longer a platform for email, it's just another platform for advertising that just so happens to have e-mail capabilities.

> There ain't no such thing as a free lunch.
> -Robert Heinlein, *The Moon Is a Harsh Mistress*

> If you're not paying for the product, you are the product.
> -Derek Powazek, web afficianado.

These two quotes here (and don't bash me on the second one, you'll understand where I'm going with it here) summarize why I started to learn more about email as a unit. GMail scans all its incoming (and presumably outgoing) emails to check for spam and malware, an innocent practice in itself, until you also realize that the content of your email, say, about your current financial portfolio, is also being scanned, leading you to get Google Ads for financial management products for like the next week, off of one single word or phrase.

Naturally, Google's caught some substantial heat for this, because due to Google's size, we as the layman could presumably assume the following:

	* Unlimited data retention
	* Easy third party access (Especially after the Cambridge Analytica affair with Facebook)
	* Other e-mail service users not having agreed to Google's terms if they send to a Google email
	* Google's ability to change polices whenever they want to whatever they want

In this day and age, sure, we can just assume that we are being turned into a serial number of possible product purchases and be okay with it, or we can take a small little stand against having our stuff analyzed by default and run our own services.

## So why start with email, and why start with Diana?

Diana is probably the "oldest" VM of the entire batch. However, when I say oldest, I mean oldest in my knowledge of how to set up its functionality and keep it running. Diana, in its many iterations, is the VM I have worked with the most in terms of tweaking and adjusting its functionality. It has gone from being a simple Exim server with limited functionality and Cyrus to a fully-blown Postfix and Dovecot instance with all kinds of goodies.

I started with the original [NSA proof your email in 2 hours](https://sealedabstract.com/code/nsa-proof-your-e-mail-in-2-hours/index.html) tutorial by Drew Crawford, in the days of Debian Wheezy, when I was just getting started out with an ancient custom junker that I made out of spare parts. It's a great starter on Postfix, Dovecot, OpenDKIM and SPF, and the MySQL stuff that you need to get the easy stuff started. And believe me, it's actually a lot easier to do than setting up PAM stuff.

Although, I started to look for a different solution as I learned there were some shortfalls that I discovered in doing this tutorial:

	* EncFS, which is suggested in this tutorial, has some [major security concerns](https://www.ict.griffith.edu.au/anthony/info/crypto/encfs.txt).
	* No support for very long filenames. Emails can potentially have long UUIDs that are associated with their file, so this is a bit of a hitch.
	* EncFS has to have the same restrictions on the source directory that it came from. While still being somewhat platform-independent, this is still a pain in the butt.

So, instead of using EncFS, I use a dedicated LUKS partition that is unencrypted at boot, because I'm more concerned with my emails being protected while the system is cold. I also haven't found an appropriate system that is about the same as EncFS that performs the same kind of protections that EncFS does, otherwise I would be using it in lieu of.

So, in my search for a different solution that I could combine what I learned from Drew's tutorial, I found Christoph Haas's astounding [ISPMail tutorial on workaround.org](https://workaround.org/ispmail/stretch) (Debian Stretch version linked). Literally it's Drew's tutorial on steroids, and I'm really pleased to have found it. I ran through it in a relatively quick fashion and was quite impressed with the level of detail put into it. However, it still has a few pitfalls, notably the inability to have send-only or SMTP-only accounts. So...I moved on.

Cue a few years and a few advancements in technology since then, and some casual perusing, I came across [a rather updated tutorial done originally in German for Ubuntu 16.04 ported to Debian](https://thomas-leister.de/en/mailserver-debian-stretch/), and was blown away. It was still relatively easy to do, but the problem was that, in my advances in knowledge and drawbacks in time, I wanted a near-fully set-it-and-forget-it solution, including manipulation of the database that would drive the entire mail server. Plus, a lot of these tutorials are based off having your own public IP that is presumably static. Ever since the university changed their networking to only hand out privates unless you're on the development network (which is truly the Internet Wild West), my experimentations have had a massive dent put in them.

We'll get to my escapades with that though in a moment.

Hopefully you're starting to see the refinement of the push-button solution I've started to come up with here. At least for myself. More lurking on the interwebs brought me to [a German-language site](https://yannici.de/server/linux/debian/debian-8-mailserver-installation/) that, while made for Debian 8, introduced me to ViMbAdmin, a very useful solution for database manipulation for your email server. In other words, a way to be lazier with more energy or efficiency.

The last site I came across that, while at this point was not directly helpful, but was of otherwise note to me was [this one](https://123qwe.com/tutorial/#how-to-use). The ISP Mail Caramel Edition tutorial was super cool, and I'd be remiss in saying the guy put in a lot of time and effort to make this work. That's where I came up with the idea that I need to have a few variations of my tutorials.

So...here we are. Introducing the ***Diana Email Project Trifecta!*** These three editions will have some deviations to them, but in order to break things up (I'm not trying to generate clicks, I swear.) I'm going to have separate posts detailing our configurations and why we're setting things the way we are.

Without further ado, here's our three projects! See which one is right for you!

	* **The Public Edition**: You have a public IP address that is both static and tied to you. You're probably paying extra in some way for it. Either way, you've got the ability to host your services without any extra effort.
	* **The Semi-Public Edition**: This one is the one that I actually used for most of my tenure while at the university, at least while they were handing out public IP addresses like they were candy. Your IP address is public, but it's dynamic, and your ISP is probably limiting the inbound and outbound ports we need for this. We'll also be taking advantage of dynamic DNS here, so keep that in mind. This one is a little bit more involved, requiring two external (but fairly trustworthy) services to help us get our email out there.
	* **The Private Edition**: Probably the most encompassing of the three, this one is where you're fully behind a private network in which you have *zero* control of the border router that does in fact have the public IP address. You're on a private address (i.e. the 10, 172.16, or 192.168) that may or may not change regularly, and port control is otherwise as strict as it can be since you're not able to be publicly routed to. This one I operated on for a short time before I closed it all down as having been successful yet a lot of effort to maintain.

I'll first have a post detailing what we do for the editions, that way you'll have all the configurations set up in advance and know they'll work.

## What do we need?

I'm a huge proponent of having "that one damn nice DigitalOcean-grade tutorial" somewhere on the Internet that lets you do a bunch of fun stuff in a short amount of time. So, for ours, here's what you'll need:

	[x] **Linux experience.** You can do this on Unix, and I have done it, but Linux is generally a little easier to use. Even if you're no expert, I'll provide the commands so you can copypaste them into an SSH session for ease of use. Plus, I'm going to go to effort to make this as platform-homogenous as I can, and will point out where you'll need to do the legwork for your platform.
	[x] **Time.** I like to be very dilligent with this work and will allocate a lot of time (and hang out with some friends in some MMOs or on Discord) while doing the work. Because I've got the resources, I will have an SSH session running, with a VMware Remote Console session ready if I really mess something up, since Jupiter, Diana's host metal, is sitting quite literally across from me.
	[x] **Something reasonably powerful for such a task.** I usually provision a Debian 9 VM with about 2 GB of RAM and 32GB of storage space. However, because Jupiter's got a LOT of space available for small VMs and has 128GB of DDR2 FBDIMM SDRAM, I've got a lot to work with.
	[x] **Patience.** I like to do this kind of stuff on a nice Friday evening with some homemade ramen, a non-alcoholic beer, and a terminal session. Your mileage may vary.

	So...without any further ado, let's get our hands wet, shall we?
