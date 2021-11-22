---
layout: post
title:  "Unknown or incorrect time zone: 'Europe/Berlin'"
permalink: /unknown-or-incorrect-time-zone-europe-berlin
date:   2019-05-11 11:18:00
tags: Databases
---
After installing a new application on one of my server, i got following error message:
{% highlight text %}
Unknown or incorrect time zone: 'Europe/Berlin'
{% endhighlight %}

Changing the locale settings didn't help

{% highlight bash %}
ahmet@testserver1:~$ mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 219
Server version: 10.1.38-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SET time_zone = 'Europe/Berlin';
ERROR 1298 (HY000): Unknown or incorrect time zone: 'Europe/Berlin'
{% endhighlight %}

The problem could be solved this way:

{% highlight bash %}
ahmet@testserver1:~$ mysql_tzinfo_to_sql /usr/share/zoneinfo/|mysql -u root mysql -p
Enter password:
Warning: Unable to load '/usr/share/zoneinfo//leap-seconds.list' as time zone. Skipping it.
{% endhighlight %}
