---
layout: post
title:  "Downgrade Mariadb"
permalink: /downgrade-mariadb
date:   2019-08-10 13:32:00
tags: Databases Mariadb Linux Server
---

Here is a short explanation on how to downgrade a minor version of MariaDB on Debian based systems.

You can find all packages and sources on the official MariaDB website: https://downloads.mariadb.org/mariadb/

In this example, we will do the downgrade from 10.4.7 to 10.4.6: https://downloads.mariadb.org/mariadb/10.4.6/
The package source can be added in parallel to existing sources in _/etc/apt/sources.list_:

{% highlight bash %}
deb [arch=amd64,ppc64el,arm64] http://mirrors.n-ix.net/mariadb/mariadb-10.4.6/repo/ubuntu/ xenial main
{% endhighlight %}

Perform downgrade:

{% highlight bash %}
apt install mariadb-server=1:10.4.6+maria~xenial mariadb-server-10.4=1:10.4.6+maria~xenial mariadb-client=1:10.4.6+maria~xenial mariadb-client-10.4=1:10.4.6+maria~xenial mariadb-server-core-10.4=1:10.4.6+maria~xenial mariadb-client-core-10.4=1:10.4.6+maria~xenial mysql-common=1:10.4.6+maria~xenial mariadb-client=1:10.4.6+maria~xenial libmariadb3=1:10.4.6+maria~xenial
{% endhighlight %}

You can block updated to newer versions by performing a _hold_ on the packages:

{% highlight bash %}
apt-mark hold mariadb-server*
{% endhighlight %}