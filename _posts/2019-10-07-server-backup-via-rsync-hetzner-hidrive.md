---
layout: post
title:  "Server Backup via rsync on Hetzner HiDrive"
permalink: /server-backup-via-rsync-hetzner-hidrive
date:   2019-07-10 06:15:00
tags: Backups Server Monitoring
---

It should not be necessary to explain the importance of backups. Here is a short article on how to perform simple backups from Debian based servers to Strato HiDrive (rsync) storage.

First of all, we need to know which directories we want to backup and 2 new directories:

{% highlight text %}
mkdir /backup
mkdir /backup_latest
{% endhighlight %}

Let's assume _/home_, _/etc_ and _/opt_ are the directories we want to safe.

This is the main script:

{% highlight text %}
#!/bin/bash

# Directory name structure
FOLDER=$(printf $(date +%Y%m%d) && printf "-" && printf $(date +%H%M%S))

# Directories to be backed up
rsync -a /home/ /backup_latest/home/
rsync -a /etc/ /backup_latest/etc/
rsync -a /opt/ /backup_latest/opt/

# You can stop services if needed
# systemctl stop <YOUR SERVICE>.service

rsync --delete -a /home/ /backup_latest/home/
rsync --delete -a /etc/ /backup_latest/etc/
rsync --delete -a /opt/ /backup_latest/opt/

# Start perviously stopped services
# systemctl start <YOUR SERVICE>.service

# Package backup
tar -zcvf /backup/$FOLDER.tar.gz /backup_latest/

# Remove backups older than 10 days
find /backup -mtime +10 -exec rm -f {} \;

# Copy files via rsync to HiDrive
# rsync --delete -avze "ssh" /backup/ meinserverbackup-userxyz@rsync.hidrive.strato.com:/users/backups-user/backups/webserver
{% endhighlight %}

The last line is commented out for testing purposes. This should be changed in production.


You should use a dedicated user for backups. After creating the user, OpenSSH Keys needs to be placed in the Settings and activate the rsync protocol:
![Strato Settings](/assets/images/Strato_Einstellungen.png)