---
layout: post
title: "Apache2, and Needlessly Strong Security"
subtitle: "The more-in-one security configuration."
date: 2019-01-20 15:15:00
background: '/img/postheads/apache.jpg'
---

# Introduction

Apache is undoubtedly the most popular web server software in use today, popular since April of 1996. Today, it serves most of the top million of websites, and has a massive amount of features that enable it to be anything from a web server to a proxy server and even a DAV server.

Today, though, the need for interoperability is becoming slowly suceeded by the need for security. Especially for those of us that are not fifteen-man teams that are paid to do this.

What I've had to do is pour through the (absolutely massive) amount of tutorials on the Internet that all go over the same 20 or 25 steps that annoy the crap out of me. If there's any one thing I hate, it's repeated advice that never changes despite the fact the subject of the advice DOES change. But to find that one interesting tidbit of advice that you never thought of...that's what I look for.

So, what I ended up doing was documenting all new information that I found, and compiled it. Today, I've got something wicked strong and what I believe is rather secure. Occasionally I still get issues with Modevasive or Modsecurity blocking something small, or something in my configuration (I'll point out where if I have issues).

So, instead of posting those same 20 or 25 steps, we're going to assume the following:

* You are above average intelligence.
* You keep your server reasonably secured.
* You do updates and upgrades regularly.

And...I'm going to post my standard (internal!) configurations. Full disclosure though, ***this is not what I'm running publicly***. Stuff you're running publicly should have a level of Opsec to it. You don't wanna tell the bad guys when you're not home.

# The apache2.conf Configuration

What I'll be doing here is listing out each option and the security implication that I have behind it. Some of them you might agree with, some of them you might not. Everyone's needs are different, and a lot of these might be for experimentation purposes.

## /etc/apache2/apache2.conf

\#\#\#\#
\#
\# BEGIN APACHE2 CONFIGURATION
\# HOST: <host>
\#
\#\#\#\#
Nothing like a little bit of text file flair, right?

\#\#\#
\#
\# STANDARD OPTIONS
\#
\#\#\#
Some more flair, for the most part.

```Mutex file:${APACHE_LOCK_DIR} default```
Standard configuration for Debian-based Apache.

```PidFile ${APACHE_PID_FILE}```
Standard configuration for Debian-based Apache.

```ServerTokens Prod```
The logic behind this is that, whenever your server sends a response, it also embeds a lot of additional information like your version, which can potentially give away your operating system.

##### Security Implication
For example, full on a Ubuntu server might look like ```Apache/2.2.8 (Ubuntu) mod_python/3.3.1 Python/2.5.2 PHP/5.2.4-2ubuntu5.7 with Suhosin-Patch mod_ssl/2.2.8 OpenSSL/0.9.8g```. That's...way too much information to be giving out passively. Setting this to Prod (or ProductionOnly if you're on a RHEL/Fedora/CentOS based system) causes the server to only output ```Server: Apache```, which is much better.

```ServerSignature Off```
This is that message you get at the bottom of an autoindex'd directory that tells you all about the server that generated the autoindex. 

##### Security Implication
This is a little too much information, and by setting this to ```Off```, we hide that information from any autoindex'd directories.

```FileETag None```
These turn off the server generating unique tags for every file that help a browser's cache update files if they are newer or missing.

##### Security Implication
The problem with these unique tags is just that: They're unique enough to be tied to a particular site. In fact, they're unique to a specific server. Particularly for virtualized or load-balanced sites, one page recieved from one server will not match the tag of the *same* page from a *different* server. So...by disabling that test, we require browsers to always load content as delivered from the site, and we keep the server from being easily identified.

```TraceEnable off```
Disables the TRACE command on Apache2.

##### Security Implication
So, this is basically a really screwed up attempt at making things easier that ends up making nothing easier and poking a huge hole in a secure site configuration. What TRACE does is that it allows you to make a sort of ping request to a server supporting, allowing you to see what is being sent and recieved between the server. By leaving TRACE on, you leave a hole in your server such that enterprising individuals (a.k.a. me and the other guys parusing the Internet casually) can use it to perform some cross-site stuff.

(There's a FABULOUS [whitepaper on TRACE](https://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf) that explains why this is a really, REALLY bad thing to have on.)

```Timeout 300```
Standard configuration for Debian-based Apache.

```KeepAlive On```
Standard configuration for Debian-based Apache.

```MaxKeepAliveRequests 100```
Standard configuration for Debian-based Apache.

```RedirectMatch 404 "^.*\/\.(?!well-known).*$"```
This keeps the server from serving any extra files that may be things like .git files or .htaccess files, pretty much anything with a dot in front of it that would be considered by many Linux-based operating systems to be "hidden". Obviously we don't wanna be serving this stuff, so let's just not.

```KeepAliveTimeout 5```
Standard configuration for Debian-based Apache.

```\# These need to be set in /etc/apache2/envvars
User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}```
Standard configuration for Debian-based Apache.

```HostnameLookups Off```
This keeps Apache from doing IP-to-FQDN lookups, which can be piecemeal operations that add up quickly. Plus, most modern servers have WHOIS on them, or you can easily browse to an Internet registry and check.

```ErrorLog ${APACHE_LOG_DIR}/error.log```
Standard configuration for Debian-based Apache.

```LogLevel warn```
Standard configuration for Debian-based Apache. You shouldn't really need to do anything fancy with this unless you've got serious issues.

```
\# Include module configuration:
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf

\# Include list of ports to listen on
Include ports.conf
```
All pretty standard stuff. I have to say, Debian breaking things up like this can be very nice sometimes.

```
<Directory />
	Options FollowSymLinks
	AllowOverride None
	Require all denied
</Directory>
```
Keeps us from serving the root directory. Standard configuration for Debian-based Apache.

```
<Directory /srv/>
	<LimitExcept GET POST HEAD OPTIONS>
	Options -Indexes -Includes FollowSymLinks
	Require all granted
	</LimitExcept>
</Directory>
```
Okay, so I might get some heat for this, but what the hey, I'm running my own methods.

You'll note here the LimitExcept directive. I see a lot of places around the Internet that say "use it, it's useful!" and others that say "don't use it, it doesn't make sense!". Really, the end result is that both sides don't know what they're talking about.

##### Security Implication
The idea is that you need to be in control of what methods you support, which by far is GET. POST is usually for when you have a site with a form or whatnot, and HEAD lets you pull some of the site content for various purposes. However, there are other methods like DELETE that REALLY don't need to be enabled. Any enterprising individual that finds a site and tests can try to interact with the site through, say, a Telnet client, and send the DELETE to wreak havoc. Needless to say this is bad, so we'll limit the commands a client can send.

You'll also note here that we have disabled Indexes and Includes on /srv/. I host multiple webroots on some servers out of the /srv/ directory because typically it is an encrypted partition on my servers that I have to unlock at boot with a key and a password.

##### Security Implication
The last thing we need is a web root that is in production having a directory without an index that lets us see the file system, and for a server that is running Includes to be including things that might make our security footing weaker. We can override this with .htaccess files if we need, so it's good to have it globally off and set it on a per-directory or per-file basis.

```
RewriteEngine On
RewriteCond %{THE_REQUEST} !HTTP/1\.1$ 
RewriteRule .* - [F]
```
These lines actually require all requests made to my servers to be in HTTP/1.1.

##### Security Implication
The reason for this is because not only is HTTP/1.0 deprececated, there's a major flaw in it with [session hijacking](https://en.wikipedia.org/wiki/Session_hijacking#History_of_HTTP).

```AccessFileName .htaccess```
Standard configuration for Debian-based Apache.

```
<FilesMatch "^\.ht">
	Require all denied
</FilesMatch>
```
Standard configuration for Debian-based Apache. Keeps people from reading .ht anything files.

```
<FilesMatch "^.*\.(css|html?|js|pdf|txt|xml|xsl|gif|ico|jpe?g|png)$">
	Order Deny,Allow
	Allow from all
	Require all granted
</FilesMatch>
```
This is kind of a cool thing that I found on the internet somewhere. Basically what we're doing here is whitelisting files that can be served by their file extension/type.

##### Security Implication
This lets us have a broad control over what files we will be serving...the last thing we need to be doing is somehow, some way, serving binary files!

```
LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent
```
Standard configuration for Debian-based Apache.

```
IncludeOptional conf-enabled/*.conf

IncludeOptional sites-enabled/*.conf
```
Standard configuration for Debian-based Apache.

```
SetOutputFilter DEFLATE
AddOutputFilterByType DEFLATE text/html text/plain text/xml text/javascript text/css text/php
SetEnvIfNoCase Request_URI .(?:exe|t?gz|zip|iso|tar|bz2|sit|rar) no-gzip dont-vary
SetEnvIfNoCase Request_URI .(?:gif|jpe?g|jpg|ico|png)  no-gzip dont-vary
SetEnvIfNoCase Request_URI .pdf no-gzip dont-vary
SetEnvIfNoCase Request_URI .flv no-gzip dont-vary
DeflateFilterNote Input instr
DeflateFilterNote Output outstr
DeflateFilterNote Ratio ratio
LogFormat '"%r" %{outstr}n/%{instr}n %{ratio}n%%' DEFLATE
CustomLog logs/deflate_log DEFLATE
```
So, this isn't really a security thing, but a very useful and powerful performance boost, and I thought I would include it here. Basically...what happens is when your webserver uses gzip encoding, you get like a 4-to-1 compression of HTML and such to reduce the costs (and bandwidth!) of serving you files.

\#\#\#
\#
\# END STANDARD OPTIONS
\#
\#\#\#
I swear, half my text files are flair.

\#\#\#
\#
\# SSL OPTIONS
\#
\#\#\#
Flair!

```SSLPassPhraseDialog builtin```
This is useful if you have a private key for your server that has a password on it. Sure, you can just remove the password from the key, but the goal is that you can use the key with the best security you can.

##### Security Implication
This directive allows you to specify either manually inputting the key password on every restart (which can get unwieldy), or specifying a file that the server can run through an ```exec:``` directive in which you echo the password (much more secure because only root should be able to read that file).

```
SSLSessionTickets On
SSLSessionCache "shmcb:/var/cache/mod_ssl/scache(512000)"
SSLSessionCacheTimeout 300
```
This is a performance booster for clients that are connecting to our Apache2 server via TLS. This particular setting makes us use a hash table that will time out clients after 300 seconds.

##### Security Implication
What this does is that it specifies that we want to use a session cache to speed up parallel requests for the same client. This also helps by allowing multiple requests to the same client to be fullfiled with the same session information, thereby reducing the number of handshakes that must be done. So by reducing the number of handshakes, we reduce the possibility that a malicious client can figure out if we're using something exploitable.

```
SSLUseStapling on
SSLStaplingResponderTimeout 5
SSLStaplingReturnResponderErrors off
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
```
This activates our OCSP stapling capability, and sets the stapling up such that we don't return responder errors (we don't want clients to know we're using it), and where to store our stapling cache information. I don't currently have a system that is able to support OCSP stapling, but I'm enabling it here for future use because it's a very useful technology to have.

##### Security Implication
Basically, OCSP stapling allows a certificate presenter (in this case, the web server) to both implement Online Certificate Status Protocol (OCSP), and creates a time-stamped OCSP response signed by the CA to verify for clients that the CA (and thereby the certificate presenter) is who they say they are. This also makes it so that clients do not always have to contact the CA to verify that the presenter is who they say they are, which is a huge drawback of OCSP.

```
\#\#\#
\# NEEDS twuewand
\# Requirements: Python 2.7+
\# https://github.com/rfinnie/twuewand.git
\#\#\#
SSLRandomSeed startup builtin
SSLRandomSeed startup file:/dev/urandom 2048
SSLRandomSeed startup exec:/usr/local/bin/twuewand 128
SSLRandomSeed connect builtin
SSLRandomSeed connect file:/dev/urandom 2048
SSLCryptoDevice builtin
```
So, this is kind of a cool thing I discovered while surfing around the Internet. It's a entropy pool addition that allows us to have an even greater amount of entropy to use while generating seeds. It's called twuewand, created by Ryan Finnie. [Here's his announcement on what it does](https://www.finnie.org/2011/09/25/introducing-twuewand/),[and here's the Github repository](https://github.com/rfinnie/twuewand).

##### Security Implication
In a VM (which a lot of my servers are), we have very limited sources of entropy, so any entropy we can get we will take. The explanation on twuewand does a great job of highlighting how Linux has three sources of randomness, and only one of them is pretty secure, so for a VM to have additional randomness (through additional tools such as haveged or rng-tools) is highly desireable because that means we will have more robust generation for cryptographic operations. In effect, what this stanza here says is that we want to start up some entropy and connect to it, allowing us to query our seeds for randomness.


```SSLCompression off```
This turns off SSL compression.

##### Security Implication
This is a big one. It prevents the [CRIME attack](https://en.wikipedia.org/wiki/CRIME).

```SSLHonorCipherOrder On```
Enforces the clients that connect to conform to the server's cipherset.

##### Security Implication
By enforcing server ciphersets rather than allowing the client to set the ciphers used, we prevent the client from potentially using an insecure cipherset that would open us up to eavesdropping, exploits, and other nastiness.

```SSLProtocol All -SSLv3 -TLSv1 -TLSv1.1 +TLSv1.2```
This defines what protocols we want to use in our crypto.

#### Security Implication
There is no civilized reason we should be offering SSLv3. Get your apps upgraded, people. And because I don't have any need for compatibility, I turn off TLSv1 and TLSv1.1, so that we only use TLSv1.2 (for now, since TLSv1.3 is now a thing).

```SSLCipherSuite DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:!aNULL:!eNULL:!LOW:!MEDIUM:!3DES:!MD5:!CAMELLIA:!EXP:!DSS:!PSK:!SEED:!ECDSA:!ADH:!IDEA:!DES```
This is the meat and potatoes of your Apache crypto solution. Everything about what your TLS is matters here, ESPECIALLY if you're requiring clients to honor the server cipher order (which we are).

#### Security Implication
The implications behind this cannot be overstated. None of what we've configured matters if we're still somehow using an IDEA or a 3DES cipher. These however are up to personal preference, your needed specifications, and whatever constraints you have such as apps that only work using certain encryption schemes. My personal preference is for perfect forward secrecy as well as AES256 for strength and a not-SHA1 hashing because at this point it's otherwise known. Take these how you will.

If you really want to get into the nitty-gritty of it, [the Wikipedia page on TLS has some very informative charts on the supported algorithms for TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security).

```SSLOptions +StrictRequire```
This *forces* forbidden access when ```SSLRequireSSL``` or ```SSLRequire``` succesfully determine that access should be denied.

##### Security Implication
This prevents someone enterprising from trying to bypass the two directives mentioned above and will prevent access if either of these conditions are met. Usually you use this in like a .htaccess file.

\#\#\#
\#
\# END SSL OPTIONS
\#
\#\#\#
End flair.

\#\#\#\#
\#
\# END APACHE2 CONFIGURATION
\# HOST: <host>
\#
\#\#\#\#
End flair!


So that was the gist of my master Apache2 configuration. I'd hope you find some interesting stuff in there, I always liked to find a change and see why that change was.

Next up, I'll detail my site-specific configurations that I use and reconfigure accordingly. I will make a note, I usually get rid of 000-default and default-ssl largely because they are unnessecary and because of my own configuration standards. If they're not used or needed, out they go.

Note: I've not included the VirtualHost definitions for sake of obviousness.

## /etc/apache2/sites-available/https.conf

\#\#\#
\#
\# HTTPS STANDARD CONFIGURATION
\# HOST: <host>
\#
\#\#\#
I guess I have a thing for document flair. Breaks up the monotony a little.

```DocumentRoot /srv/http/```
Like I said, I usually have the ```/srv/``` directory on my VMs a encrypted partition that requires unlock at boot. I have different configuration files for if I have multiple sites, but when I have only one site, it usually resides in ```http/```. All my sites are accessed over HTTPS as well unless there's a genuine need for them to not be HTTPS. Plus, call me crazy, but I feel like this is way less typing when I'm going around in the filesystem.

```
ErrorLog /srv/https-error.log
CustomLog /srv/https-access.log combined
```
Having a different place for the logs to go allows for you to do some useful things. In this case, I'm putting the logs on my encrypted partition for safekeeping, and for ease of access for some of my tools.
	
```DirectoryIndex index.php```
Defining the DirectoryIndex allows you to have some control over what is expected to be the index.

##### Security Implication
You can nest these in Directory directives in order to control navigability.

```
<Directory />
	Options None
	AllowOverride None
	Require all granted
</Directory>
```
Even though we already defined the default site root security in the apache2.conf, we always define it again just in case I make a mistake (which does in fact occur) and unset a security permission on accident. Additionally, we can override on a case-by-case basis with .htaccess files.

```
RewriteEngine On
RewriteCond %{THE_REQUEST} !HTTP/1\.1$
RewriteRule .* - [F]
```
I duplicated the RewriteEngine rules here so that way if I have a VirtualHost that is having issues I can turn it off globally and still have it running on all my other VirtualHosts while I figure out where the problem is.

```
SSLEngine on
SSLCertificateFile	/etc/apache2/ssl/web.cert.pem
SSLCertificateKeyFile /etc/apache2/ssl/web.key.pem
```
We have the SSLEngine on here because we defined this site for mod_ssl. All of the certificates and keys are actually distributed from my CA, Terminus. This helps me manage which sites I have in operation.

```
SSLProtocol +TLSv1.2
SSLHonorCipherOrder On
```
We define these again in order to be doubly sure of our settings.


```Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"```
We're getting to some header definitions now. This particular setting activates HTTP Strict Transport Security, which helps protect against protocol downgrade attacks and cookie hijacking.

##### Security Implication
This means that a site demands that it only be interacted with over HTTPS and never via HTTP. Used in conjunction with a HTTP redirecting host, this allows for us to require all connections to be performed over HTTPS.

```Header always set X-Frame-Options "SAMEORIGIN,DENY"```
This header setting does a variety of things. X-Frame-Options can be used to indicate a browser's behavior for opening pages in frames or iframes, prevening content that has been unintentionally (or intentionally) embedded into the page from being opened.

##### Security Implication
Here, we're setting it such that a page will display if it is the same origin as the page itself (SAMEORIGIN), and prevent all other pages from being displayed in a frame or iframe (DENY). A third option is available as a sort of whitelist, ALLOW-FROM, which when supplied with a URI will allow a page to be displayed only from that origin.

```Header always set X-Content-Type-Options "nosniff"```
This setting sends to the browser the demand that it not attempt to interpret files delivered using MIME-type sniffing.

##### Security Implication
Content sniffing is used to compensate for a lack of metadata, basically. It helps a browser figure out what it will be rendering and shifts its resources accordingly. The problem with this is it creates an enormous exploit because a file can easily lie and say it's one thing, and the server delivers other content that enables exploitation of the client. We want to set it such that the client doesn't try to sniff so that way if we DO have something bad, the client can't get hit by this route.

```Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure```
This option controls how cookies are set and accessed by the browser.

##### Security Implication
By setting Set-Cookie in this manner, we assure that all cookies are not accessible to client-side scripts (HttpOnly), and that they must be delievered over HTTPS (secure). If you have a cookie that has to be delivered over HTTP (because it has some arcane programming or something), you can whitelist that cookie by adding a ```Header edit Set-Cookie``` followed by the cookie's identification in proper definition.

```Header set X-XSS-Protection "1; mode=block"```
This setting indicates to a browser to perform XSS protection, which will implement a series of XSS protections against some basic attacks.

##### Security Implication
This one is a no-brainer. There are four values: disabled (0), enabled with sanitizing (1), enabled with rendering prevention (1; mode=block), and enabled with report (1;report=<reporturl>). I don't have a report facility, so we'll set it to enabled with render prevention.

```Header set Referrer-Policy: "same-origin"```
This header controls our referrer policy.

###### Security Implication
There's actually a variety of possible settings for this, but I picked 'same-origin' because it's going to be mostly me accessing these services.

```Header set Content-Security-Policy "default-src 'self'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self'; frame-ancestors 'self'"```
This header sets our content security policy such that the things that the browser can run can only originate from the page they came from.

##### Security Implication
By setting these the way they are, we can prevent some XSS, Clickjacking, and other code injection stuff that has been put into our trusted webpages. We trust the stuff we put on there, but who knows, the stuff that our stuff references could get jacked up.

```Header always unset X-Powered-By```
Unsets the Powered By header.

##### Security Implication
I have really NO idea why this header is even a thing. Basically what it does is it tell what the server is running. It can be easily manipulated too. But at the end of the day it's just extraneous information, so we can turn it off.

```SSLOpenSSLConfCmd DHParameters "/etc/apache2/ssl/dhparams.pem"```
This directive is for where we put our dhparameters that we need for our cipherlist. Your DH parameters have no real reason to be below 4096 in today's age.

```SSLUseStapling on```
I like to expicitly mention options again just in case I forgot something somewhere else. Like I said before, I just need the OCSP stapling facility and boom, I'm set up.

```SSLSessionTickets Off```
Suprisingly I have a few issues with an app or two that is dated and SSLSessionTickets. I leave it in here in case I need to actually have it off, usually it's commented out.

```AllowEncodedSlashes NoDecode```
Small help feature that keeps slashes from being decoded as being something other than what they are: slashes.

```		
SSLProxyEngine Off
SSLProxyVerify None
SSLProxyCheckPeerCN Off
SSLProxyCheckPeerName Off
```
I use these four lines in case I have an application that, for one reason or another, forces me to use Apache as a proxy to it. Sometimes these apps run through Caddy or whatnot, so having the ability to just turn on SSL proxying helps a lot. Of course, I still have to define the ProxyPass and ProxyPassReverse, so it's just removing a single step.

\#\#\#
\#
\# END STANDARD CONFIGURATION
\# HOST: <host>
\#
\#\#\#
And end flair.

This next one should be pretty easy. It's just a HTTP site that 301's to the HTTPS site.

## /etc/apache2/http.conf

```
	Redirect permanent / https://<site>.<domain>/

	ErrorLog /srv/http-error.log
	CustomLog /srv/http-access.log combined
```
That's it. That's all there is.

# TL;DR
If you'd like to have a relatively decent configuration that you can easily move around via whatever deployment method you use, I'm more than happy to share!

### /etc/apache2/apache2.conf

```
\#\#\#\#
\#
\# BEGIN APACHE2 CONFIGURATION
\# HOST: <host>
\#
\#\#\#\#

\#\#\#
\#
\# STANDARD OPTIONS
\#
\#\#\#

Mutex file:${APACHE_LOCK_DIR} default

PidFile ${APACHE_PID_FILE}

ServerTokens Prod
ServerSignature Off
FileETag None
TraceEnable off
Timeout 300
KeepAlive On
MaxKeepAliveRequests 100
RedirectMatch 404 "^.*\/\.(?!well-known).*$"
KeepAliveTimeout 5

\# These need to be set in /etc/apache2/envvars
User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}

HostnameLookups Off
This keeps Apache from doing IP-to-FQDN lookups, which can be piecemeal operations that add up quickly. Plus, most modern servers have WHOIS on them, or you can easily browse to an Internet registry and check.

ErrorLog ${APACHE_LOG_DIR}/error.log

LogLevel warn

\# Include module configuration:
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf

\# Include list of ports to listen on
Include ports.conf


<Directory />
	Options FollowSymLinks
	AllowOverride None
	Require all denied
</Directory>

<Directory (yourwebroot)>
	<LimitExcept GET POST HEAD OPTIONS>
	Options -Indexes -Includes FollowSymLinks
	Require all granted
	</LimitExcept>
</Directory>

RewriteEngine On
RewriteCond %{THE_REQUEST} !HTTP/1\.1$ 
RewriteRule .* - [F]

AccessFileName .htaccess

<FilesMatch "^\.ht">
	Require all denied
</FilesMatch>

<FilesMatch "^.*\.(css|html?|js|pdf|txt|xml|xsl|gif|ico|jpe?g|png)$">
	Order Deny,Allow
	Allow from all
	Require all granted
</FilesMatch>

LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent

IncludeOptional conf-enabled/*.conf

IncludeOptional sites-enabled/*.conf

SetOutputFilter DEFLATE
AddOutputFilterByType DEFLATE text/html text/plain text/xml text/javascript text/css text/php
SetEnvIfNoCase Request_URI .(?:exe|t?gz|zip|iso|tar|bz2|sit|rar) no-gzip dont-vary
SetEnvIfNoCase Request_URI .(?:gif|jpe?g|jpg|ico|png)  no-gzip dont-vary
SetEnvIfNoCase Request_URI .pdf no-gzip dont-vary
SetEnvIfNoCase Request_URI .flv no-gzip dont-vary
DeflateFilterNote Input instr
DeflateFilterNote Output outstr
DeflateFilterNote Ratio ratio
LogFormat '"%r" %{outstr}n/%{instr}n %{ratio}n%%' DEFLATE
CustomLog logs/deflate_log DEFLATE

\#\#\#
\#
\# END STANDARD OPTIONS
\#
\#\#\#

\#\#\#
\#
\# SSL OPTIONS
\#
\#\#\#

SSLPassPhraseDialog builtin

SSLSessionTickets On
SSLSessionCache "shmcb:/var/cache/mod_ssl/scache(512000)"
SSLSessionCacheTimeout 300

SSLUseStapling on
SSLStaplingResponderTimeout 5
SSLStaplingReturnResponderErrors off
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"

\#\#\#
\# NEEDS twuewand
\# Requirements: Python 2.7+
\# https://github.com/rfinnie/twuewand.git
\#\#\#
SSLRandomSeed startup builtin
SSLRandomSeed startup file:/dev/urandom 2048
SSLRandomSeed startup exec:/usr/local/bin/twuewand 128
SSLRandomSeed connect builtin
SSLRandomSeed connect file:/dev/urandom 2048
SSLCryptoDevice builtin

SSLCompression off
SSLHonorCipherOrder On
SSLProtocol All -SSLv3 -TLSv1 -TLSv1.1 +TLSv1.2
SSLCipherSuite DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:!aNULL:!eNULL:!LOW:!MEDIUM:!3DES:!MD5:!CAMELLIA:!EXP:!DSS:!PSK:!SEED:!ECDSA:!ADH:!IDEA:!DES
SSLOptions +StrictRequire

\#\#\#
\#
\# END SSL OPTIONS
\#
\#\#\#
End flair.

\#\#\#\#
\#
\# END APACHE2 CONFIGURATION
\# HOST: <host>
\#
\#\#\#\#
```

## /etc/apache2/sites-available/https.conf
```
\#\#\#
\#
\# HTTPS STANDARD CONFIGURATION
\# HOST: <host>
\#
\#\#\#
I guess I have a thing for document flair. Breaks up the monotony a little.

DocumentRoot (yourwebroot)

<Directory />
	Options None
	AllowOverride None
	Require all granted
</Directory>

RewriteEngine On
RewriteCond %{THE_REQUEST} !HTTP/1\.1$
RewriteRule .* - [F]

SSLEngine on
SSLCertificateFile	/etc/apache2/ssl/web.cert.pem
SSLCertificateKeyFile /etc/apache2/ssl/web.key.pem

SSLProtocol +TLSv1.2
SSLHonorCipherOrder On

Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
Header always set X-Frame-Options "SAMEORIGIN,DENY"
Header always set X-Content-Type-Options "nosniff"
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
Header set X-XSS-Protection "1; mode=block"
Header set Referrer-Policy: "same-origin"
Header set Content-Security-Policy "default-src 'self'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self'; frame-ancestors 'self'"
Header always unset X-Powered-By

SSLOpenSSLConfCmd DHParameters "/etc/apache2/ssl/dhparams.pem"
SSLUseStapling on
SSLSessionTickets Off

AllowEncodedSlashes NoDecode
	
SSLProxyEngine Off
SSLProxyVerify None
SSLProxyCheckPeerCN Off
SSLProxyCheckPeerName Off


\#\#\#
\#
\# END STANDARD CONFIGURATION
\# HOST: <host>
\#
\#\#\#
```

## /etc/apache2/http.conf

```
	Redirect permanent / https://<site>.<domain>/
```

# Conclusion

Well...I hope you learned a lot from this. I'll probably be coming back to edit this one as things come along...however, when I start to do some stuff with TLSv1.3 and HTTP/2.0, I'll probably end up making a new post. Also...you might notice that I let out a whole bunch of IE stuff. If you're using IE, you need to use something else, seriously.