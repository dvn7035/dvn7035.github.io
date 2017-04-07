---
layout: post
title: Setting up a Web Server
description: "Setting Up a Web Server"
excerpt_separator: <!--more-->
modified: 2017-03-09
tags: [infrastructure@localhost, HTTP, web, server]
categories: [operations]
---

Web servers are nothing special. They're just machines that respond to specially crafted
messages that follow the HTTP standard. The last <a href="{{ site.baseurl }}{% post_url
2017-01-04-crash_course_networking %}#HTTP">page </a> had an example of what these kind of messages
can look like, but I encourage you to read up some more about HTTP. Setting one up wasn't difficult,
so here's how I did it.  
<!--more-->

So first off, I started by taking advantage of the [GitHub Student Developer
Pack](https://education.github.com/pack). At the time, I did not have the hardware to self host this
site in my home, so I resorted to using a virtual private server instance from DigitalOcean. It was 
a no brainer since they were offering $50 in credit.  

After signing up, I started an Ubuntu 16.04 machine with my SSH keys preloaded. SSH is but another
protocol that runs on port 22 which allows you to control a computer remotely. Using a key is 
generally safer than using a password and DigitalOcean saves me the trouble of setting up (and doing 
a write up for) an SSH server...for now.  

![digital_ocean_1]({{ site.url }}/images/digital_ocean_1.png)  
###### *Ubuntu? What a noob.* ######  
![digital_ocean_2]({{ site.url }}/images/digital_ocean_2.png)
  
![digital_ocean_3]({{ site.url }}/images/digital_ocean_3.png)
  

At this point, I signed up for a domain with Google Domains. Any registar would have sufficed; in
fact in the Github Student Developer Pack, they offer a free .me domain through Namecheap. Here I
made two DNS records: an A record that maps my domain to the ip address of my DigitalOcean box and a
CNAME record that maps www to dvn7035.com.  

![google_domains_dns]({{ site.url }}/images/google_domains_dns.png)  

If you're not clear on DNS, it's just a way for clients like you to visit my web site without having
to put an IP address in your URL bar. The first record with the '@' symbol is an exact match. Anyone
who types in exactly "dvn7035.com" will be given the ip address listed. The second record with "www"
means that visitors who prepend "dvn7035.com" with "www." (i.e. "www.dvn7035.com", though seriously
who does this?) will be pointed to dvn7035.com which surprise, surprise gets handled by the first
record.  

Next I SSH'd into my machine  
```
ssh -l root dvn7035.com
```

# Installing Nginx #  
Now it was the time to install and Nginx (pronounced: "engine x") which is a web server program that
hosts my site. My web pages are mostly static, that is, the page see you now is the same file
somewhere on this server. Nginx will look into a certain folder and give you this page based on your 
url.  

Before I could install Nginx, I had to add the software's repository to my list of repositories. In
/etc/apt/sources.list I added these two lines  
```
deb http://nginx.org/packages/ubuntu/ xenial nginx  
deb-src http://nginx.org/packages/ubuntu/ xenial nginx  
```

This allowed me to install the most recent nginx version for Ubuntu 16.04 from Nginx themselves
instead of relying on the packages in the official Ubuntu repository  

I then ran these commands to import Nginx's PGP so that apt, short for Advanced Package Tool, won't
complain that the repository has unverified signatures.  
```
wget http://nginx.org/keys/nginx_signing.key  
apt-key add nginx_signing.key  
apt update  
```

To actually install Nginx it was as simple as typing  
apt install nginx  

# Configuring Firewall #  
After installation however, the web site was still not yet available. This was because iptables, the
firewall for Linux was blocking incoming connections. I typed the following commands to allow web
traffic through 80 (HTTP), 443 (HTTPS), and 22 (SSH). Also I set the default behavior to block
incoming connections not specified by a specific rule.  
```
apt install ufw  
ufw default deny incoming  
ufw enable  
ufw allow ssh  
ufw allow http  
```

You might have noticed I installed and used a program called ufw. Ufw is a easier front-end for
iptables because at the time I did not know how to write iptable rules directly. Anyway, I then
confirmed the rules by running  
```
ufw status  
```

# Configuring Nginx #  
At this point, when I visited my website through the browser, I was greeeted with the default Nginx
homepage. That was nice and all, but there were some configurations that I wanted in place like url
redirection and HTTPS. The first thing to check was the /etc/nginx directory. In general, /etc holds
all configuration files for any application. The first file of interest in this directory was
nginx.conf.  

The most imporant line then was  
```
include /etc/nginx/conf.d/\*.conf;  
```

This meant that nginx will look for all files ending with ".conf" in /etc/nginx/conf.d for
configuration variables as well. By looking at that directory  
```
ls /etc/nginx/conf.d  
```

I had found default.conf. Opening it with my favorite text editor in vim, I was able to change it to
this  
```
server {  
    listen 80;  
    server_name www.dvn7035.com dvn7035.com;  
    return 301 https://www.dvn7035.com$request_uri;  
}  
  
server {  
    listen 443 ssl;  
    server_name dvn7035.com;  
    ssl_certificate /etc/letsencrypt/live/dvn7035.com/fullchain.pem;  
    ssl_certificate_key /etc/letsencrypt/live/dvn7035.com/privkey.pem;  
    return 301 https://www.dvn7035.com$request_uri;  
}  
  
server {  
    listen       443 ssl;  
    server_name  www.dvn7035.com;  
    ssl_certificate /etc/letsencrypt/live/dvn7035.com/fullchain.pem;  
    ssl_certificate_key /etc/letsencrypt/live/dvn7035.com/privkey.pem;  
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;  
    ssl_prefer_server_ciphers on;  
    ssl_dhparam /etc/ssl/certs/dhparam.pem;  
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';  
    ssl_session_timeout 1d;  
    ssl_session_cache shared:SSL:50m;  
    ssl_stapling on;  
    ssl_stapling_verify on;  
    add_header Strict-Transport-Security max-age=15768000;  
  
    location / {  
        root   /usr/share/nginx/html;  
        index  index.html index.htm;  
    }  
    error_page   500 502 503 504  /50x.html;  
    location = /50x.html {  
        root   /usr/share/nginx/html;  
    }  
}  
```
  
This all might look a little daunting, but let me break it down some before your eyes gloss over it:  
The first block tells Nginx to listen in at port 80 and respond to any request with a redirect to
the secure version the site at "https://www.dvn7035.com". Notice that even if you didn't prepend the
url with www, the site with still redirect you to the canonical name that has the www prefix.  

The second block deals with HTTPS requests. Nginx will listen in on port 443, and redirect any HTTPS
request without the www. You might ask, what's with the strict adherance to have www for my website,
but unfortunately the reasons are kinda esotreic and border on epistemology. Just understand that it
really just my preference.

Finally, the third block is where the real meat of the configuration lies. Notice that it specifies,
ssl_certificate and ssl_certificate_key. These are files needed to serve the web site over HTTPS.
They aren't there yet, but I know these files will exist once I run an application called
Certbot (I've rebuilt this site too many times to count). Also the field ssl_ciphers specify the
encryption standards I'm allowing the server to accept. I have to confess that a lot of these
settings are taken from the DigitalOcean tutorial and according to Symantec's CryptoReport, these
settings make the site vulnerable to BEAST attacks, oops. I'll fix these issues when I containerize
Nginx. Anyway, the "location /" sub-block specifies the web root of the web site. This means Nginx
will look in this folder when you specify a specific page. For example, if you typed
"https://www.dvn7035.com/blah", Nginx will look into the folder /usr/share/nginx/html for a file
called blah (or blah.html, I think). If you don't specify a specific page and just type
"https://www.dvn7035.com" the server will give you /usr/share/nginx/html/index.html or (htm).  

# Getting HTTPS #  
I was nearly done, all I had to do next was install Certbot and run the program to grab an SSL
certificate from Let's Encrypt. Let's Encrypt is a certificate authority that gives out free SSL
certificates for web site owners if they can prove that they own a domain. This is done through
the ACME protocol which Certbot handles for me.  

I added Cerbot's repository,  
```
add-apt-repository ppa:certbot/certbot  
```

instaled it,  
```
sudo apt-get update  
sudo apt-get install certbot  
```

and then ran it.  
```
certbot certonly --webroot -w /usr/share/nginx/html -d dvn7035.com -d www.dvn7035.com.com  
```

The certificates are only valid for like 3 months. After that, they become invalid and your browser
will probably warn you that this web site is unsecure. I currently rerun this command every so often
even though I should just have a cron job automate it for me. Oh well, I'll do it next time when I
migrate the site again.

All that was left then was to restart Nginx.  
```
service nginx restart
```
