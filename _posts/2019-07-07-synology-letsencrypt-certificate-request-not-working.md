---
layout: post
title:  "Synology LetsEncrypt certificate request not working"
permalink: /synology-letsencrypt-certificate-request-not-working
date:   2019-07-07 13:00:00
tags: Synology Server Linux Certificates Letsencrypt
---

Synology is offering automatic LetsEncrypt certificate requests since a while now. This service is working most of the time great. Unfortunately, some of my certificates could not be requested lately. 
![Synology Certificates Red](/assets/images/Bild1.png)
Requesting the certificates manually resulted in the following error:
![Synology Certificate Error](/assets/images/Bild1.png)

Error message: _No connection to LetsEncrypt could be established. Check if the domain name is valid._

After some try and error I found out that the request didn't work because of some reverse proxy rules i set before. Everything worked fine again after removing the reverse proxy rules and requesting the certificates again. 

There is no easier solution at the moment. Synology is already informed about this bug.