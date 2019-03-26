---
layout: post
title: "Diana, Part Two: Starting From Scratch"
subtitle: "Everyone starts somewhere."
date: 2018-12-22 13:30:00
background: '/img/postheads/diana.png'
---

# Initial Setup

So, for this, we're going to assume you have a completely blank slate. A old standby PC, a new VM, whatever. And we're going to assume that, for the time being, you're using an updated (read: civilized) version of your favorite distro.

For this setup though, if you want reliability and stability, Choose Debian (tm)! Debian's my personal favorite, so for all of the setup, we'll be thinking in the context of Debian.

Now, to make a note: I have a series of scripts that are mantained on my TFTP server, Unxia, that help me get things set up. For this setup (and in general), I'd like to recommend a few things that you use.

 * [Byobu](http://byobu.co/) is a terminal window multiplexor. Think Screen and Tmux but on steroids. Being able to fork off and split a terminal window into two without having to fight all your windows if you're in a GUI, or cycle through all your VTYs if you're on the command line. I'd like to point out this thing is INVALUABLE over SSH, as it lets you have all the same splitting functionalities that you would have if you were physically on the terminal.

 * [Linux_logo](http://www.deater.net/weave/vmwprod/linux_logo/) is a text-based logo generator that you can do a lot of fun stuff with. Usually, what I have it do is generate for me some basic system info including the current load of the system and output it for me on the login screen. Mostly for semantics, I like having really pretty looking login screens that are also functional.

 * [Figlet](http://www.figlet.org/) for honorable mention here, I use it mostly for giving out a cool-looking name whenever I login and have my motd displayed. Not much to say about it here, it's pretty straightforward.

 * **bash-completion** because I don't install with all the default options.

So, when you're doing your fresh installation, take a few notes here:

 [x] **Consider installing with no other features.** Like when tasksel offers if you wanna install a mailserver or an OpenSSH server. Don't bother with that stuff, we'll be overwriting their configurations later anyhow. You're building a specialized appliance here, let's keep the crap to a minimum.
 [x] **When performing partition setup, you will need a partition that is set up for crypto.** Our guide assumes that you will be setting up your encrypted partition to be mounted under ```/srv```. The idea behind this is that we are worried about our email being safe at rest, rather than while in use. This means that, where applicable, you'll need to set up a *partition for physical encryption* or whatever your distro calls a LUKS partition.
 [x] **You may want to take a look at some of my other posts for other needed security configurations.** We're installing primarily an email solution, but we'd be remiss without having a method of remote access and a way to present our webmail. In particular, you might want to consult [my SSH post](https://natethesage.github.io/2018/12/21/ssh-security-and-you.html) and/or [my Apache2 post](https://natethesage.github.io/2019/01/20/apache2-security-and-why-it-matters.html).

# Configuration Options

So, now we're going to go into the three configuration options here. They range from straightforward to "thinking outside of the box by removing the box from the equation entirely". Like I detailed previously, here are the three options:

* **The Public Edition**: You have a public IP address that is both static and tied to you.
* **The Semi-Public Edition**: Your IP address is public, but it's dynamic, and your ISP is probably limiting the inbound and outbound ports we need for this.
* **The Private Edition**: You're on a private address that may or may not change regularly, and port control is otherwise as strict as it can be since you're not able to be publicly routed to.
