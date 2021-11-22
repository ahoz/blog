---
layout: post
title:  "Automate Wildcard LetsEncrypt Certificate Requests on Hetzner Cloud"
permalink: /automate-wildcard-letsencrypt-certificate-requests-on-hetzner-cloud
date:   2020-06-23 08:00:00
tags: Letsencrypt Python Server Certificates
---

Lets create a script for automated LetsEncrypt wildcard certificate retrievement.

The workflow and scripts on this page currently works only for Hetzner domains or domains which nameservers are hosted on Hetzner.

## Install Certbot and prepare config

Certbot installation

{% highlight text %}
ahmet@server1:~$ apt install certbot
{% endhighlight %}

Clone repository

{% highlight text %}
ahmet@server1:~$ git clone https://github.com/ahoz/hetzner_dns_letsencrypt_auto_wildcard.git
ahmet@server1:~$ cd hetzner_dns_letsencrypt_auto_wildcard/
ahmet@server1:~/hetzner_dns_letsencrypt_auto_wildcard/$ chmod +x hetznerdnshook.py
{% endhighlight %}

Move config.ini.example to config.ini and set Hetzner DNS token. You can find the token on the DNS page of Hetzner cloud, then Manage API Tokens.

![Hetzner Cloud DNS](/assets/images/001.png)

Enter a token name and click on create access token.

![Hetzner Cloud Create Access Token ](/assets/images/002.png)
![Hetzner Cloud Access Tokens](/assets/images/003.png)

The token will only be shown once. Copy it to your clipboard and set it in your config.ini.

## Request certificate

### First request

The first request requires a mail address and an approval of LetsEncrypt conditions and data protection declarations. 

{% highlight text %}
ahmet@server1:~/hetzner_dns_letsencrypt_auto_wildcard/$ sudo certbot certonly --manual --preferred-challenges dns --manual-auth-hook ./hetznerdnshook.py -d domain.de -d *.domain.de
{% endhighlight %}

### Future requests

The mail address and approval is not needed for future requests.

{% highlight text %}
ahmet@server1:~/hetzner_dns_letsencrypt_auto_wildcard/$ sudo certbot certonly  --manual --preferred-challenges dns --manual-auth-hook ./hetznerdnshook.py -d domain.de -d *.domain.de --agree-tos  --manual-public-ip-logging-ok
{% endhighlight %}

Do not forget to replace domain.de and *.domain.de with your domains.

## Old ACME token removals

You can remove old ACME tokens with this script:

{% highlight text %}
ahmet@server1:~/hetzner_dns_letsencrypt_auto_wildcard/$ python3 hetznerdnshook.py --delete donmain.de
{% endhighlight %}