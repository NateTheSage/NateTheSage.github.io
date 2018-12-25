---
layout: post
title: "SSHd Security And You"
subtitle: "Making your server paranoid strong and damn secure."
date: 2018-12-21 13:30:00
background: '/img/postheads/02.jpg'
---

# So what is this?

So, this will be one of a couple lead-in posts to some future posts that will deal with my (what I like to believe are) super-secure services.

In today's modern day and age, we need secure remote access to servers for a variety of reasons. The Internet is a stew of untrusted users and machines, nation-states are going at one another over cyberspace, and our refrigerators and thermostats are rising up and taking over the Internet. (Okay, not really, but with Mirai still floating around, is it so different?)

Nearly fifty years ago, Telnet ('**Tel**etype **net**work') was the choice of most typical remote access methods. Of course, fifty years ago, the Internet was still largely a closed-system network, only made up of trusted entities, such as government, universities, and some companies. It allowed for remote command-line interface access, but the complete plaintext of the communications today reduces Telnet to being nothing more than a web browser. In fact, for a while, I used Telnet to check the weather in Tulsa at [rainmaker.wunderground.com](telnet://rainmaker.wunderground.com) (Telnet link!), I'm not sure if it's still up but I believe it is. I encourage you to try it, Google "places to telnet" and look around!

Today, SSH is nearly ubiquitous in use, and it has a lot of cool modern features like 2FA and keying management. We're going to talk about some of the cooler stuff you can do with an SSH server, and some important security considerations to have.

# SSH Server Considerations

To start, we're going to be considering two different kinds of SSH services: One for Windows, and one for literally everything else.

[OpenSSH](https://www.openssh.com/) is the foremost example for literally everything else. Originally developed from the free SSH program that was initially coded by Tatu Ylönen, OpenSSH had its original release in 1999 and today is developed as a part of OpenBSD. What many may consider one program is actually a series of feature sets that serve as alternatives to both Telnet and FTP, integrated into most modern Linux and Unix operating systems, and derivatives thereof.

Recently, as of October 19, 2015, Microsoft has announced native support for OpenSSH via PowerShell. I've gotten the chance to play with it, and I have to say, while it is feature-similar, it's not quite true OpenSSH. The alternative that is my personal favorite that I'm about to propose is not exactly true OpenSSH either, but it's got great integration into some Windows stuff, particularly Remote Desktop.

[Bitvise SSH Server](https://www.bitvise.com/ssh-server) and [Bitvise SSH Client](https://www.bitvise.com/ssh-client) (both priorly referred to as WinSSHD and Tunnelier) are awesome programs that I use in my environment for Windows hosts. The SSH Server is great because it allows you to have access to a GUI control program that lets you manipulate the server settings in case you need to do so remotely, and the SSH Client has a one-touch Remote Desktop button that lets you tunnel a RDP session over SSH, adding both some additional security and making it harder for someone to man-in-the-middle your RDP session.

The considerations I give for OpenSSH will also be given optionally for WinSSHD and Tunnelier.

# OpenSSHD Hardening

A lot of our hardening comes from [Secure Secure Shell](https://stribika.github.io/2015/01/04/secure-secure-shell.html) as it is a very useful source of explanation as to why even stribika does some of the stuff with SSH he does.

## Key Exchange

Our first thing that we're going to have to change is our key exchange. The key exchange protocols suffer from NIST having their hand in the cookie jar, a weak DH modulus, and SHA1, which is now otherwise known since Google demonstrated a SHA1 collision attack.

Given these things, that gives us the last 2. We'll keep these for later.

```
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
```

I highly recommend having both Curve25519 and DHGE in that snippet both on, because I'm not aware of Bitvise's client being able to do Curve25519 yet. We'll get to that.

Since we're enabling DHGE, we should generate some fresh moduli.

```
# rm /etc/ssh/moduli
# ssh-keygen -G /etc/ssh/moduli.all -b 4096
# ssh-keygen -T /etc/ssh/moduli.safe -f /etc/ssh/moduli.all
# mv /etc/ssh/moduli.safe /etc/ssh/moduli
# rm /etc/ssh/moduli.all
```

This should give you some fresh moduli to work with that have been created with some good entropy. Presumably. You might worry if your entropy is good. If you've got that worry, you may have other problems this guide isn't meant to help you with.

## Authentication

### Server Keying

I have two different considerations (that only one matters now currently because of networking) for my network access. Internally, we want it to be as fast as possible, but externally, we want it to be as secure as possible.

I used to use both ED25519 keys and RSA keys to authenticate to my SSH servers, but now I use largely RSA keys as I am not worried about the speed because I cannot perform remote access from outside my internal network.

**Please note, we will not be using RSA keys below size 1024, and we will not be using DSA keys period!! We have to assume the NSA has the capability to read these!**

If you're interested in the differences between ED25519 and RSA, there's a pretty briefly-informative post about it [on Medium.](https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54)

```
Protocol 2
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key
```

So we're now explicitly saying that SSH protocol 2 is going to be used. We should always be explicit with our orders. Also, we're going to get rid of the default keys because I don't really trust the keys that are generated on installation.

A note here. I've made the iterations for ED25519 and the keysize for RSA ambiguous here because I like to go over the top for my security. People may give me a hard time about it, but as I'm the only person accessing them, I like there to be needlessly strong security.

```
# rm ssh_host_*
# ssh-keygen -t ed25519 -a <rounds> -f ssh_host_ed25519_key -N "" < /dev/null
# ssh-keygen -t rsa -b <size> -f ssh_host_rsa_key -N "" < /dev/null
```

I like to name my keys rsa_superkey and ed25519_goodkey, but you could name then whatever you want, these are snippets from an automation setup script I have.

### Client Keying (Optional if you don't have OpenSSH client in use)

Passwords suck. Passwords generated by humans suck even more. So, as a prime requirement when it comes to SSH nowadays, we're going to get some keying setup for clients.

# ssh-keygen -t ed25519 -a <rounds>
# ssh-keygen -t rsa -b <size>

You'll want here to be able to choose (or verify) where your keys are being saved, and set passwords, as well as see your key's random imageart to make sure it's unique.

Now, once you've gotten your user set up such that you can authenticate via password, we're going to turn it off.

In ```/etc/ssh/sshd_config```:

```
PasswordAuthentication no
ChallengeResponseAuthentication no
PubkeyAuthentication yes
```

### Optional: TOTP for SSH

Borrowed from the EXCELLENT multi-platform tutorial [at Linux.com.](https://www.linux.com/blog/securing-ssh-two-factor-authentication-using-google-authenticator)

So for something kind of cool and additionally fun, we can add TOTP for SSH, giving us 2FA.

I've borrowed most of this [from DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-ubuntu-16-04) which purveys the most AMAZING tutorials that are both informative and easy to follow.

For this, you will need an OTP application. My personal favorite, and one that I highly recommend, is FreeOTP for both [iOS](https://itunes.apple.com/us/app/freeotp-authenticator/id872559395?mt=8) and [Android](https://play.google.com/store/apps/details?id=org.fedorahosted.freeotp&hl=en_US), although for the Android one I recommend the [F-Droid](https://f-droid.org/en/packages/org.fedorahosted.freeotp/) variant.

Next, let's get this thing installed.

```
# apt install libpam-google-authenticator
```

Then let's run it to get it set up.

```
# google-authenticator
```

The library will now prompt you for a lot of questions. We want to say yes to the following:

	1. Yes - time-based authentication tokens. Codes will change randomly after a certain amount of time elapses rather than going through a sequence.

You'll see a bunch of stuff fly by, this is normal. If you're in a GUI, you'll probably see a QR code printed onto your screen, if not, you may have to manually input the key. After that though, we'll be configuring the PAM's functionality.	

	2. Yes - update our google-authenticator files. If you say no, the program quits and nothing happens.
	3. Yes - disallow multiple uses of the same token. Makes all codes expire after use.
	4. No - short time window. Keeps tokens good for 30 seconds to account for clock skew.
	5. Yes - rate limiting for authentication attempts. No more than 3 every 30 seconds.
	
Alright, it's configured. Now, if you want, you can take the .google-authenticator file and use it on other systems.

Now, we have to configure our sshd's PAM sources.

```
# nano /etc/pam.d/sshd
```

And edit the bottom of the file:

```
# Standard Un*x authentication.
#@include common-auth
auth required pam_google_authenticator.so nullok
```

Now, we restart our SSH service and then make our SSH daemon aware of our MFA system.

```
# nano /etc/ssh/sshd_config
```

Find UsePam at the bottom of the file, and add this:

```
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

And just like that, we're done! Restart one last time, and we're good.

### Optional: THREE factor authentication, cause why not!

Edit sshd's PAM sources again!

```
# nano /etc/pam.d/sshd
```

And edit the bottom of the file:

```
# Standard Un*x authentication.
@include common-auth
auth required pam_google_authenticator.so nullok
```

Done! Now SSH will require password, publickey, and the keyboard-interactive token!

If you're more interested in the TOTP, particularly the recovery, consult the DigitalOcean tutorial.

## Authorized Users and Ciphers

### Authorized Users

Next, we're going to create an SSH group that is authorized for logging in. Users that are not a part of this group will not even be considered for login authentication if they're not on it.

```
# nano /etc/ssh/sshd_config
```

Wherever you like in the file:

```
AllowGroups ssh
```

And now whenever you want to append a user into the new group:

```
# usermod -a -G ssh <username>
```

### Ciphers

To summarize the ciphers that stribika used, and for compatibility with Bitvise SSH Client (in case you like it enough to switch from PuTTY and SCP), here's what we'll use:

```
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
```

That's it. That's all we need.

## MACs

Most importantly, our message authentication codes. The most you need to know from this is:

```
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

You have to have the hmac-sha2-256-etm@openssh.com line in there if you want Bitvise to connect to OpenSSH, it's the only HMAC that Bitvise supports that's not suckage.

# OpenSSH Server TL;DR

```

###
#
# BEGIN SSH CONFIG
# HOST: <yourhost>
#
###

### SERVICE SETUP
Protocol 2
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key
AddressFamily inet
ListenAddress 0.0.0.0
Port <customport>

### AUTHTYPES
PasswordAuthentication no
ChallengeResponseAuthentication no
PubkeyAuthentication yes
GSSAPIAuthentication no
HostbasedAuthentication no

### TRANSMISSION SECURITY SETTINGS
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
Compression yes

### ALLOWED GROUPS AND USERS
AllowGroups ssh
AllowUsers <users separated by spaces>

### ADDITIONAL SECURITY SETTINGS
X11Forwarding no
AllowAgentForwarding no
PermitRootLogin no
AllowTcpForwarding no
AllowStreamLocalForwarding no
GatewayPorts no
PermitTunnel no
PermitEmptyPasswords no
MaxAuthTries 3
IgnoreRhosts yes
UseDNS no

### COMPLIANCE SETTINGS
LogLevel VERBOSE
AuthorizedKeysFile /etc/ssh/authorized-keys/%u
Banner /etc/issue.net
ClientAliveInterval 360
ClientAliveCountMax 0

### PAM SETTINGS
UsePAM yes
AuthenticationMethods publickey,password publickey,keyboard-interactive

### SFTP SUBSYSTEM
# Log sftp level file access (read/write/etc.) that would not be easily logged otherwise.
Subsystem sftp  /usr/lib/ssh/sftp-server -f AUTHPRIV -l INFO

###
#
# END SSH CONFIG
#
###
```

# Bitvise SSH Server Hardening

So, when we first get WinSSHD installed, generally it will generate a 3072 bit RSA key and 384 bit ECDSA key, as well as a certificate. We're going to clear all that.

We'll be doing everything through the Bitvise SSH Server Administration Panel. Have no fear! Everything is easy to find.

Click **Manage host keys** and **remove** on both the keys. We'll regenerate some new ones. Generate a **new 4096-bit RSA key** and **set it as the employed keypair**. Do the same for **Ed25519**.

Now, once you're done with that, click **OK** and go down to the Certificates. Click **Manage Certificates** and remove the certificate that was generated during installation. These certificates are for **FTPS and not SFTP or SSH**. We're not going to be using FTPS, and will be disabling it soon anyway.

We'll now configure Bitvise to have similar settings to OpenSSH. Click **Edit advanced settings**.

First things we're going to configure are in the Server settings. Click **Server**, and look at your right-hand window. The settings you'll need to configure to are listed here:

	[x] Omit server version
	[x] Omit relative directories in SFTP
	
We want to hide our server version because this could indicate potential attacks an adversary could perform, and omitting relative directories is useful because it keeps us from trying to copy the entire parent directory when doing a file transfer, a real issue with some other SSH and SFTP clients.

**(Optional)** Remove the IPv6 listening address :: in the **IPv6** Bindings window.

Another quick note. In the IPv4 window, you have the option to, when editing the 0.0.0.0 rule, add **SSH Obfuscation** and **PROXY protocol settings** as well as **UPnP**. Generally I don't edit these because while SSH Obfusication is awesome, not many clients support it, not many clients need PROXY protocol support, and advertising your SSH server over UPnP ***KIND OF*** defeats the entire purpose.

The usage statistics are super nice to have, basically it's a fancy way to say "logging".

Now we're getting to the more interesting stuff. **SSH Algorithms** are next.

You should want your **key exchange** window to look like this:

	[x] Curve25519
	[ ] *all the ecdh-sha2 entries*
	DH Group Exchange: minimum size 4096
	DH Group Exchange: maximum size 4096
	[x] diffie-hellman-gex-sha256
	[ ] diffie-hellman-gex-sha1
	[ ] *all the diffie-hellman group 16, 15, 14, and 1 sizes*
	
The next window, **signature**, should look like this:

	[x] Ed25519
	[ ] *all the ecdh-sha2 entries*
	[x] rsa-sha2-512
	[ ] rsa-sha2-256
	[x] ssh-rsa
	Minimum RSA key size (bits): 4096
	[ ] ssh-dss
	
Your **encryption** window should look like this:

	[x] aes256-gcm
	[ ] aes128-gcm
	[x] aes256-ctr
	[ ] aes192-ctr
	[ ] aes128-ctr
	[ ] 3des-ctr
	[ ] aes256-cbc
	[ ] aes192-cbc
	[ ] aes128-cbc
	[ ] 3des-cbc
	
This next one's easy. **Data integrity protection** should look like this:

	[x] hmac-sha2-256
	[ ] hmac-sha1
	
**(Optional)** If you'd like to configure compression (which I recommend), have it look like this:

	[x] zlib
	[x] none
	[x] Delay compression
	
**TLS Algorithms** you can leave alone, since we're not running any FTPS.

**Session** has a few things we should look at.

**(Optional)** The **Banner message file** gives you a capability to display a banner not unlike on OpenSSH. This is generally required for compliance, but can also be a fun way for you to know it's your server with a little ASCII art.

Since we're in here, we should **Limit total sessions** to **All** and change **Maximum total sessions** to **5**.

If you'd like to know anything more about the Bitvise SSH Server options, those little question marks next to each option give great information on what each option does, and recommendations.

Make sure to restart your WinSSHD service from the panel, then give it a try!

# Conclusion

This is about the gist of what I do for SSHds. I like to believe this adds some great security, especially if the server is Internet-facing, which mine aren't anymore, they all hide behind a private network.

I'll write a follow-up post soon on what I do for clients typically, although I will admit that my SSH experience has turned to using Bitvise over Wine, that's how much I like it.

~Nathan