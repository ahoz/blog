---
layout: post
title:  "Apt fix fatal failed to fork"
permalink: /apt-fix-fatal-failed-to-fork
date:   2019-08-07 06:25:00
tags: Server Linux
---

I got the following error message while trying to update one of my recently created test servers:

{% highlight text %}
ahmet@testserver1:~$ apt update && apt dist-upgrade
Hit:1 http://mirror.hetzner.de/ubuntu/packages bionic InRelease
Hit:2 http://mirror.hetzner.de/ubuntu/packages bionic-updates InRelease
Hit:3 http://mirror.hetzner.de/ubuntu/packages bionic-backports InRelease
Hit:4 http://mirror.hetzner.de/ubuntu/packages bionic-security InRelease
Hit:5 http://ppa.launchpad.net/certbot/certbot/ubuntu bionic InRelease
Hit:6 http://archive.ubuntu.com/ubuntu bionic InRelease
Hit:7 http://security.ubuntu.com/ubuntu bionic-security InRelease
Hit:8 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
Hit:9 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
Reading package lists... Done
FATAL -> Failed to fork.
{% endhighlight %}

After some research I found out, that low memory led to this issue. The update process worked fine after stopping the applications.