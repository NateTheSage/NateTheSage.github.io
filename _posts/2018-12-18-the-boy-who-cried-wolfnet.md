---
layout: post
title: "The Boy Who Cried WolfNet"
subtitle: "Now that I've got your attention..."
date: 2018-12-18 11:45:00
background: '/img/postheads/01.png'
---

# The First Real Post About The Network

So, what I first want to talk about is the network. The one thing that I'm proud of that has grown from a little router with two PCs on it, to a rather large internal network that hosts a type 2 hypervisor, a packet filtering platform, and a host of research, work, and in one instance gaming, PCs and peripherals.

[Here's a picture of the network.](/img/posts/01_wolfnet.png)

<div class="thumbnail">
      <a href="/img/posts/01_wolfnet.png">
        <img src="/img/posts/01_wolfnet.png" alt="WolfNet Network Schematic" style="width:100%">
        <div class="caption">
          <p>WolfNet as it appears at the time of this posting.</p>
        </div>
</a>

So...to give some color, this entire network is a private network behind a private network. (Inceptive, no?) Essentially, the University of Tulsa took the smart (read: slightly inconveniencing) step of converting the entirety of their networks to private IP address schemes. Originally, the university handed out public IPs to every single computer that connected to it, mostly due to the university maintaining possession of nearly all of a class B space.

This was EXCEPTIONALLY convenient for me, due to the fact that because everyone got a public IP, access inbound and outbound to the Internet was almost unfettered. I was able to host a lot of things via dynamic DNS (which, I know, is a bad thing) and was able to really begin to understand the importance of securing your border. Of course, this changed after a summer, and I came back to find I had no Internet connectivity. I threw a single switch inside my PFSense box configuration, and my Internet was back. Even though services would be harder.

After a little while though, I learned how to do SSH gateway ports, and used that for a pretty long time. If IT ever looked, they would have seen needlessly strong SSH sessions flowing back and forth from a development network IP address (that DID in fact have a public IP) that I was administrating. This worked great, although having to maintain the other host became such a pain that I ultimately abandoned the idea, despite it still working and me still having the VM that managed the ports available. All because I knew that there was pretty much no way IT would ever give a student a public IP even if it was for a good reason.

So...here's the network as it stands.

1. Networks

	1. local.net, the 10.10.10.0/27 network that most of my physical devices actually reside on. This net has very high scrutiny on the PfSense box of what comes in and out. Make using Steam and Discord kind of a PITA, but usually I have to disable a rule or add a supression and everything works out fine.
	
	2. virtual.net, the 10.20.20.0/24 network that almost all of my VMs reside on. This network only has one device connected to it, an old Dell Precision T7400, one of two, I pulled out of a dumpster near the computer science building. Suprisingly, one of them was still in working order, so I scrapped the other for parts, cleaned it up, gave it a cool OSHA tape job, and put about $400 of new hardware into it. Now if I could pull a 1250W UPS out of the dumpster that can support the 1000W power supply, that would be awesome...
	
	3. wireless.net, the 10.30.30.0/28 network that most of my wireless devices use. Interestingly, I learned how to make my Netgear R8000 get an IP from DHCP on the PFSense box and be useable while still allowing other devices to connect to it as an access point. Mostly my devices that I can't give a physical connection due to their design or other limitations.
	
Pretty cool, right? Each network uses the Unbound DNS Resolver and the ISC DHCPD servers on the PFSense box. Each host on the network sends its hostname with DHCP in order to register in the DNS resolver. So, on top of an awesome organization scheme, I have full internal DNS!

So, now I'll describe some of the device on the network. Keep in mind that I administrate with pride each and every single one of them, so I've got quite a few stories.

2. Devices

	1. local.net
	
		1. hyperzero.local.net
		
			This is my Alienware 14, a fairly moderate mistake of a computer I bought from Dell right after they bought Alienware. A friend and I at the time bought said we were going to buy one, and we made our purchases within a month of each other. That difference in time was apparently enough that he got the older (arguably better) Alienware, and I got the newer Dell-branded Alienware.
			
			My complaints about this thing are varied, but these are mostly the big ones:
			
			* The design is terrible. Heat pools in the bottom of the unit despite me keeping the fans clear and even setting the thing on many iterations of exhaust fans.
			* The battery connector died almost EXACTLY the day after the warranty was up. As the laptop was pretty much stationary at this point because it's a freaking furnace on my thighs, this didn't really bother me, I just offset it with an APC UPS that still proudly serves to this day. Only issue is that the last BIOS update I need requires the battery to be plugged in...
			* The Alienware Control Center is proprietary as crap and is very unreliable. The lights behave very wierdly, and I'm not even sure games with AlienFX API access even know how the API works themselves. Plus, some of us just want our LEDs to burn friggin' white.
			* The unit is so sensitive to electrical states that the UPS is also doing the task of power line conditioning, or what one might consider to be rudimentary power line conditioning. Even the slightest static can cause the dedicated 765M to logically disconnect from the PC. The only way to cure it is to turn the unit off, remove the battery (if your connecter still even works!), and hold the power button for exactly 20 seconds. Turn it back on, and you're golden.
			* It's a consumer computer. If I haven't established yet that most consumer computing equipment is complete garbage, let the record reflect.
			
			So yeah. When I ultimately build a new PC, this thing will become my emulator unit that I use in like a MAME device or something.
			
		2. ultrium.local.net
		
			Now this thing here I'm pretty proud of. It's a Panasonic CF-19 MK6, pretty much the most steriotypical MDC (that's Mobile Data Computer for those of you not familiar) used in government, law enforcement, etc.
			There's a fun story behind this one.
			
			This unit actually was purchased probably around 2013 or so, by the Tulsa Fire Department. This unit is fully loaded, all options, bells, and whistles included. Another friend and I hoped they got a good deal on them, because a single unit this fully loaded would have been about $8000 brand new.
			
			We bid really hard for these units at the Tulsa Surplus Auction in around November of 2017, but didn't get them...at least initially. A guy by the name of Dylan there actually got most of the units, and we got to talking to him. We learned we were all computer geeks, and that he mostly did auction turnover, with most of his turnover going to other departments, schools, and even some colleges. He sold us a few units for a little bit higher price than he bid for them, and that's how I got my probably most favorite, most portable, and coolest talking point computer ever.
			
			Here's just some of the features:
			
			* Fingerprint reader (that is super insecure if you use the Windows fingerprint logon option, Premium feature)
			* Dual-touch screen with digitizer capability (Premium feature, it's a Wacom transciever)
			* Water- and dust-resistant touchpad and keyboard with backlight (Premium feature)
			* Integrated camera and flash unit (Premium feature)
			* 4G cellular radio, CDMA capability (We think they had a government service provider contract with Verizon, Premium feature)
			* Serial-line GPS card (Premium feature, is really cool and works well!)
			
			Probaby the coolest part about these things though is the unit-on hours displayed in the BIOS. My unit was most likely used in either an engine or a chief's vehicle, because it has nearly 40k hours on it. It's like an odometer for a computer!
			
			A year later, we bought bought two base stations that were being auctioned off, and suprise suprise, just as rugged as the devices they were meant for! They work great and makes having the devices so much better.
			
			I'll detail these later, this thing was such an adventure that it deserves its own post.
			
		3. rinzler.local.net
		
			This is a Dell PowerEdge 2950 Generation III that I've recently acquired to learn more about working on a proper server platform. It even has its old DRAC5 card with it. However, it's been difficult to get set up as I can't find a lot of the drivers for it...the upgraded board has no Service Tag that comes with most Dell units, so I can only make wild guesses as to what it contains.
			
			Still, it's a pretty good server that I plan on using as a more versatile VM host, for specialized functions that ideally would require a physical hard drive connected to the VM.
			
			The most cool tweak I made to it is managing to install a super cheap SSD with a PCI mount in the rear, connected to the board via SATA (you know, the ones they say are meant for tape drives but magically work with any other drive), and a single Molex power connection clear from the front of the case to the back.
			
			[Here's the hack.](/img/posts/01_rinzlerhack.jpg)
			
			This one has it's own story too! I'll detail it in another post.
			
		4. seraphina.local.net
		
			This is "The Trashmaster", a computer that is 100% assembled from parts found either in the trash, in backrooms, or with parts that are EOL. There's 2 hard drives, about 500GB total, 24GB of RAM, as well as an old i7 950 with a board that has failing PCIe ports on it. Not joking, if you plug in 3 devices into the same bus, the CPU panics and you have to force reboot it to have none of them work. Remove one, same thing happens, except the bus is back to normal. It's a finnicky thing.
			
			I've used a Wacom Intuos Pro 6 as a mouse for it to try the Linux support, and I have to say, it is phenomenal. The level of support for it regardless of platform is amazing.
			
	2. wireless.net
	
		1. iforge.wireless.net
		
			So I think Android is really cool, and I'm an Apple convert after I started stressing over trying to jailbreak my iDevices and being just generally tired of Apple being overly friendly with problematic devices. People that are willing to help you at Apple have not been true tech support personnel. At least, that's my personal opinion.
			
			More to the point, this is a Samsung Galaxy Tab III 8.0 that runs LineageOS 15.1, GT-N5110. If you want a super nice tablet that's not branded and you can do just about anything with, get this tablet. The tempered glass protector and Otter case I got for it were so worth it, especially having the second-generation S Pen. Granted, I had to chase down a LOT of apps from the old Cyanogenmod days to make some of the LineageOS features work, but it was all worth it in the end. It lets me browse quickly and play a few guilty pleasures.
			
		2. starbaseone.wireless.net
		
			This is a Netgear Nighthawk X6 that, astoundingly is a nearly $300 router that I bought at a pawn shop for $80 just because you "can't use all channels" or something absurd like that. The only thing I remember was that I was terribly confused as to why they bought it, clearly whoever bought and whoever sold it didn't understand how Wi-Fi worked, let alone the router. I opened the (original!) box to find an otherwise still brand-new router sitting there, complete with DD-WRT support.
			
			This router is just plain awesome. I'm able to hide under the university's bands (1, 5, 6, and 11, can you believe it? The crosstalk is awful!) and have incredibly clear Wi-Fi throughout my small apartment. I just wish I could turn off the SSID, but my 3DS won't connect to hidden SSIDs even if you specify them. Oh well.
			
	3. virtual.net
	
		1. jupiter.virtual.net
		
			Of course I'm going to save the best for last and talk about my type 2 hypervisor. This is an old (and I do mean old) Dell Precision T7400 with 128GB of DDR2 FBDIMM ECC RAM, 2 Xeon E3-5530s, and seven (last I counted) hard drives all spread out with different configurations because some hard drives are being passed through directly to some VMs. This thing is what I consider the apex of my ability to build junkers and make them work again.
			
			I pulled this thing out of a dumpster, and found another one in there, still with power supplies attached! (They both work fabulously.) I was about to pull out the other unit when I realized there was no way it was going to operate again with that enormous melted hole in the motherboard. So, late at night on campus, I was sitting there in that dumpster salvaging the other unit because I'm a small guy, and to get the one out of there was impressive in itself because it was nearly 40 pounds deadweight. I tried to do it as quickly as possible because there was no way Campus Security was going to believe any story I told them about what the heck I was doing in a dumpster, regardless of my student status. Somehow I got the working unit and the scrapped one's parts out, without a hitch.
			
			I spent about a week cleaning (read: powerwashing with 98% isopropyl) the good unit, taking the case completely apart, cleaning out every nook and cranny just in case there was more dumpster juice in there than it smelled like there was, and reassembled it with a cool OSHA tape job that actually is holding the case together. I checked every slot on that unit's RAM (even pulled the riser cards!), checked both 1Kw power supplies, and put about $400 into the old computer. Seeing that computer power on and recognize all the new equipment that I put into it was the most exciting thing I've ever been a part of.
			
			This one's another that is going to get its own post (probably a few!) detailing everything I've done with it.
			
That's it! Everything I've ever learned or done here has been put into this pride and joy of a network. I hoped you found this post interesting and informative. Stay tuned, I'll have some more detailed posts coming soon.