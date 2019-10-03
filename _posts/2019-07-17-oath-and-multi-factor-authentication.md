---
layout: post
title: "OATH and Multi-factor Authentication"
subtitle: "It's our token authentication post."
date: 2019-10-03 10:30:00
background: '/img/postheads/2fa.jpg'
---

# MFA, Done The Safe Way

A problem we have come to recognize nowadays is "we have too many services".

Have you ever sat back and looked at how many things you're signed up for or participate in? I can tell you, I never thought about it
until I started to run my own email service. I got into the (somewhat useful) idea of making an alias for every service I use actively. As it turns out, I have already 200+ aliases for various odds and ends. Had I not also started using a password manager, trying to secure all these services would have been a nightmare.

Fortunately, many of the services support multi-factor authentication. In today's world of breaches and exposures, having something keeping an actor from being able to walk willy-nilly into your account.

![Modern hacker breaking into an account without MFA, 2019, colorized.](https://natethesage.github.io/img/posts/walkrightin.jpg)

Like a hangover recovery, the only thing preventing a hacker from obtaining your password from the latest database dump of hash is time. The time it takes to break the hash and get your password (or in some notable exceptions, ahem, Facebook) you use on n+1 other accounts will become your only saving grace.
You just have to hope whoever got the latest dump has something on the order of a Raspberry Pi. (Spoiler alert: They probably have some big iron.)

With the ease it takes nowadays to break into a system and stay there, I've taken it upon myself to start experimenting with MFA in more local scenarios. And sure, I could have used Google Authenticator. In fact, I did, for a very, VERY brief spell, until I discovered something strange (and, frankly, downright worrisome.) having to deal with how Google stores the secrets.

I don't know where it was, but I saw someone had inadvertently put the full URI to their Google Authenticator token, which was located on a Google-hosted resource. On a whim, I copied the URI and browsed to it. Sure enough, I was presented with their secret key in the form of a QR code. After the fact, I realized when the wiki page was last edited! In 2017!

I thought, "It's 2019! How long does Google keep these things around?!"

Immediately I started looking for alternatives. And found there really aren't many. Google made it so push-button easy now everyone just uses their PAM library.

Well, here, today, I'm presenting you an opportunity to generate these things for yourself. 100% locally sourced OTP generation!

The majority of this guide will focus on generation for \*nix. I will also touch on a solution I figured out for Windows involving a paid program but seems to be the only way involved to make this happen, plus it makes you dependent on the program's, shall we say, workability in your environment because of how it works and where it installs.

## What do we need?

For doing this on any \*nix system, we'll need the following. You should be able to get most of these things from your package manager:

* **oathtool** ~ Oathtool allows you to get the next token from a particular seed. What we'll be using it for here is to ensure we got our token set up right.

* **libpam-oath** ~ This is the PAM plugin granting us the ability to authenticate using the tokens. We're going to do some awesome stuff with this involving authentication!

* **fim** ~ (Optional) If you're solely on command line, you'll need something to pipe a QR code to your screen with. Fim does just that.

NOTE: If you're on CentOS, I haven't found something like fim, so you may need to spin up a Python HTTP server to browse and view your QR code. It's `python -m http.server` for Python3 out of your current directory.

* **qrencode** ~ QREncode will help us generate a QR code for your mobile devices to scan and process the token. This is also beneficial as if you lose your tokens on your mobile device somehow (i.e. botched updates), you can keep your token PNGs handy.

* ***Root shell access*** ~ We'll be `sudo -i` for this because we don't want to accidentally lock ourselves out.

## Step 1: Setup

***WARNING!*** Once you start, you have to follow through with this procedure. Failure to do so ***WILL*** result in you being locked out, and requiring a rescue boot to fix the changes.

Start with your root shell.

`$ sudo -i`

![Our interactive sudo shell.](https://natethesage.github.io/img/posts/otp_1.png)

Next, what we want to do is get some entropy to make into a seed. This is a hexadecimal number, unique for each user, in order for the user to have a token. We'll make two for now, for our user and for root. I piped these into files we can call them easier. It's your choice on whether or not you hang onto these when we're done.

`# head -10 /dev/urandom | sha512sum | cut -b 1-30`

![Generating our seeds.](https://natethesage.github.io/img/posts/otp_2.png)

###### What are we doing here?

`head -10 /dev/urandom` ~ We want the first 10 bytes of some random output from /dev/urandom...

`sha512sum` ~ ...and we want to make a hash of it using SHA512...

`cut -b 1-30` ~ ...and output the first 30 columns of characters...

`>> rootsecret` ~ ...into a file called 'rootsecret'.

Generally, at this point, I like to check the tokens to see how they look.

![Looks like we got some good tokens.](https://natethesage.github.io/img/posts/otp_3.png)

Next, we'll be generating our user token storage in `/etc/users.oath`.

`# touch /etc/users.oath && chmod 400 /etc/users.oath`

It is crucial nobody else is able to read this file because it will contain our plaintext secrets.

Once you've created this file, edit it using your favorite editor. I'll use `nano`.

`nano /etc/users.oath`

Once you've created it, we'll be copying things in like so. Having done my research, these are the fields I found libpam-oath to expect.

```
# This is the users file for Oath. All entries must be structured like the following:
# <token type>  <username>  <PIN>  <token key>  <counter> <failcounter> <last OTP used successfully>  <time of last OTP in localtime> <last IP used>
```
####### What are all these fields?

* *Token Type* ~ This is the kind of token we'll be using. The field is formatted to expect `ALGORITHM[/COUNTERINFO[/DIGITS]]`, with everything in brackets optional. Two types are supported.

> HOTP (What we're using, RFC 4226 compliant tokens.)
> MOTP (Mobile-OTP. I've not tried to use this, but I believe it's for older phones. You can find it [here](http://motp.sourceforge.net/).)

We'll use HOTP. The default for HOTP is assumed to be 'HOTP/E', which we don't want. This is an event-driven token, we want a time-based token in order to have a more secure token. Our token will be 'HOTP/TNN', where NN is the number of seconds in one time interval. I highly recommend 30 in order to have sufficient digit changes and to make it harder for someone to use your token who shouldn't be using it.

Finally, for digits, we'll want to use 6 digits since 6 is a pretty standard token length. So, put it all together...

`HOTP/T30/6`

* *User Name* ~ This is the Unix name of the user logging in, in other words, the 'user' part of your 'user@hostname' string in Bash.

* *PIN* ~ This is totally optional, and allows you to require a PIN be entered *before* the token is input. For example, if you set a pin of '0123' and the token rolled happens to be '456789', you would enter '0123456789'.  This is not recommended generally because this is a static PIN. If you wanted a dynamic PIN, you'd have to have it verified via a `OTPAuthPINAuthProvider`. This is out of scope for this tutorial because it appears to involve systems like LDAP or AD, however, if you absolutely want to have one, the values available are the following.

> - (No PIN.)
> + (PIN is set by a PIN provider.)
> <value> (Where <value> is a four digit number you enter.)

* *Token Key* ~ This is where our key for our tokens will reside.

* *Counter/Offset* ~ This indicates the next expected value (for event tokens) or offset (for time tokens).

* *Failure Counter* ~ This hold the amount of wrong OTPs provided by this user.

* *Last OTP* ~ This is the last good OTP provided to libpam-oath.

* *Time* ~ This field indicates the time, in local time, when the last OTP was generated.

* *Last IP Address* ~ The last IP address which entered a valid OTP token.

Now, most of these fields we will not need. The ones we're interested in are the first four.

To easily get our tokens into the file, do the following:

`# cat rootsecret >> /etc/users.oath`
`# cat usersecret >> /etc/users.oath`
`# chmod 400 /etc/users.oath`

![Putting our tokens into the users.oath file.](https://natethesage.github.io/img/posts/otp_4.png)

(Mine doesn't show /etc/users.oath because my oathtokens are already active, and I'd sooner not accidentally give those out!)

Now, let's edit the file and add in the fields we'll need.

`nano /etc/users.oath`

We'll add in these lines.

```
HOTP/T30/6  root  - <therootsecret>
HOTP/T30/6  <youruser>  - <theusersecret>
```
Where <therootsecret> is your generated root secret, <youruser> is your username, and <theusersecret> is the users's generated secret.

![Our users.oath file.](https://natethesage.github.io/img/posts/otp_5.png)

Step 1 is now complete!

## Step 2: Hooking Up libpam-oath

The next step involves telling the various PAM plugins we have an additional authentication subsystem we want to consult. In this case, we'll be heading into the pam.d directory, the home of all the PAM module configurations.

`cd /etc/pam.d`

![A typical pam.d directory's contents. I'm running GNOME as well as a few other things.](https://natethesage.github.io/img/posts/otp_6.png)

This is where your options really start to broaden a little. To keep with the basics, we'll first start with addressing three points of interest.

1. *login* ~ login addresses the getty process exists on all VTYs. (You know, the prompt you see to login.)
2. *su* ~ su, or "*substitute user*", allows you to initiate (or when given a dash, login) a shell as a different user.
3. *sudo* ~ sudo, or "*superuser do*", allows you to run processes under alternate credentials with additional fine-grained restrictions on what users can run.

This is of course if you're on a headless console. On one with X or Wayland, you'll need to edit either `gdm-password` for GNOME, or `lightdm` for LightDM. Your mileage may vary if you're using a different configuration or know what you're doing.

To edit more than one of them, format your edit command like this:

`nano /etc/pam.d/{su,sudo,login}`

Now, if you're not familiar with how a PAM file is formatted, we'll go through it really quickly here.

**realm control module [arguments]**

* *Realm* ~ PAM has four realms of management: *auth*, *account*, *password*, and *session*. We'll be inserting pam-oath into the *auth* realm.
* *Control* ~ PAM uses this field to decide what to do should a particular module pass or fail. Because we're interested in proper multifactor authentication, we will set this field to **required**.
* *Module* ~ This is the name of the module. The modules for the most part go into `/lib/security/pam`, so on Debian you are free to just type the name without the path.
* *Arguments* ~ We will want additional arguments here to indicate the users file, the time window, and the number of digits.

So, at the end of this, our line looks like so:

`auth required  pam_oath.so usersfile=/etc/users.oath window=30 digits=6`

![An example of sudo's PAM file with our entry in it.](https://natethesage.github.io/img/posts/otp_7.png)

Now, with sudo, there's a bit of a caveat. If you want to make it so we require ***root's*** token rather than the user's token (which they will be prompted for in addition to their password), you can use `visudo` and add the line `Defaults rootpw` under all the other defaults. This will require both root's password and root's token rather than the user's password and the user's token. Personally, I do this because you have to supply the root token and password with `su` anyhow.

###### Optional: SSHd

If you'd like to also secure your SSH daemon, you can add in the above line to `sshd`. Do note though if you make this change, you will need to make the following changes.

```
ChallengeResponseAuthentication yes
AuthenticationMethods keyboard-interactive
UsePAM yes
```

(If you use public keys, change `AuthenticationMethods` instead to `publickey,keyboard-interactive`. Or, if you want to go for THREE factor authentication, include `PasswordAuthentication yes` and make sure you do not include `password` in your `AuthenticationMethods`, otherwise you will be prompted twice for your password.)

## Step 3: Storing Our Tokens

For this last step, we'll need `qrencode` and `fim`. Please note if you're doing this over SSH, `fim` will not work correctly and you'll need to SFTP or SCP the QR codes to a machine where you can view images.

Now we've gotten our secrets made, and our changes to the PAM modules done, it's now time for us to store our secrets.

Run oathtool like this so you can see if your token generates properly.

```
oathtool -d6 -v \`cat usersecret\`
oathtool -d6 -v \`cat rootsecret\`
```
![Our example tokens have generated correctly!](https://natethesage.github.io/img/posts/otp_8.png)

Perfect! Now comes the interesting part. We'll now build QR codes based upon these tokens for importing into a token vault.

If you need a vault, I recommend the following:

* [Aegis](https://f-droid.org/en/packages/com.beemdevelopment.aegis/) ~ Great for Android phones with fingerprint readers. If you don't have one, it's still great and useable.
* [Authenticator](https://flathub.org/apps/details/com.github.bilelmoussaoui.Authenticator) ~ Great for Linux environments with window managers. Unfortunately its the one app I have which demands to use the light GNOME background no matter what.
* [WinAuth](https://winauth.github.io/winauth/) ~ Great for Windows environments. The client can even handle Battle.net or Steam Authenticator tokens.
* Oathtool! The very command you just used can be used to get OTPs for any secrets you have locally available!

We'll generate the QR codes here with the following command, now we know both our Base32 secrets.

```
qrencode -o root.png 'otpauth://totp/<user>@<hostname>?secret=<thesecret>'
```

Where <user> is your username, <hostname> is the hostname of the \*nix host, and <thesecret> is the Base32 secret. I haven't come up with a good way to get the Base32 secret out of oathtool yet, though, so you'll have to manually type it in.

![Our PNGs generated.](https://natethesage.github.io/img/posts/otp_9.png)

Now, if you're on a proper console session (I'm in a window manager so my results will look different), you can use the following to see the QR code and scan it with Aegis (or your OTP vault of choice).

`fim root.png`
`fim <user>.png`

Scan it in, and you're done!

## Step 5: Try It!

Now, log out. Here comes the hard part. When you log back in, you'll be prompted for your password, then you'll get a line that says:

`One-time password (OTP) for <user>: `

Enter the six-digit code from your password vault, and you should be granted access.

![An example of the prompt when using sudo -i.](https://natethesage.github.io/img/posts/otp_10.png)

## Conclusion

I hope that was quick and easy! I intend to be able to make a script for this to automate the process, but for now, this is a good exercise in learning about security while enhancing our own local security.

Thanks for reading, and I hope you learned something new!

###### Resources Used
