---
layout: post
title: "Aurora: Making a Personal CA"
subtitle: "Organizing your various snake oils."
date: 2019-01-20 15:15:00
background: '/img/postheads/05.jpg'
---

*Note: Terminus has been decommissioned as my CA, as Aurora replaced it due to a serious misconfiguration that required migration.*

Nowadays, you can never be too careful travelling where there is no encryption or protection for your daily activites. Many sites now are offering HTTPS, and many browsers now have either default preference for HTTPS, or have addons that enforce HTTPS wherever possible. But what about your internal network? Your internal services? Just because they're on the inside doesn't mean they still don't need to be kept safe like they're on the outside.

Now, I'm sure you'll be asking: Why is he just not using Let's Encrypt? Well...I suppose there's a convoluted answer to that.

*The short answer:* Let's Encrypt was well trusted, nor was it really around to begin with, when I started doing a lot of this stuff. So, I had to come up with my own methodology, which involved using OpenSSL as a CA.

*The long answer:* I started doing this kind of research a long time ago, like I said, when Let's Encrypt was not fully trusted still by most browsers. So, I did a lot of experiments with GnuTLS and OpenSSL doing a lot of CA stuff, like learning how the PKI system worked at a very fundamental level. Once I started to get into this more, I came across [Jamie Nyugen's tutorial](https://jamielinux.com/docs/openssl-certificate-authority/) on how to get started with a CA on the command line, and actually built one and ran it this way for a while.

The only problem one runs into with doing a CA on the command line is that it gets tedious and you add a bunch of extra steps just to get your own certificate that is signed by your own CA and intermediate CA. So I started to look for a web-based solution that I could run on its own dedicated VM (I had gained the current infrastructure I have today by this time) that would be offline until it was required.

My search on the Internet revealed somewhat disappointing results. To be expected, I'm sure. Not everyone willingly runs a CA web interface, even on their internal networks for fear of it being discovered and being used. But, I did come across one application that thankfully was a pretty-well self-contained PHP application: PHPki. There's lots of variants, however, I recommend the variant that [limefamily has forked](https://github.com/limefamily/PHPki-Digital-Certificate-Authority). It mostly works and with only a few additional settings gives you a reasonably secure setup.

*Note: I set up [a repository](https://github.com/NateTheSage/phpki) if you're interested in using my tweaks. At the time of this posting, it doesn't work that great, but I'm fixing that.*

If you want to be able to have your own local CA (your browsers will yell at you still but you'll have a heirarchy), you can follow these steps:

 1. You'll need PHP5.6 to run PHPki, at least some of the older versions. PHP7.0 ***WILL*** break things with it. Your OpenSSL can be pretty modern though, as far as I understand.

 `sudo apt install php5 apache2 openssl && service apache2 stop`
 
 2. Install Apache2 with a ***basic HTTP-configured site***. We're going to make it HTTPS later.
 3. Point Apache2 to your site root. For this tutorial, we will assume it's at ```/srv/http```.
 4.
