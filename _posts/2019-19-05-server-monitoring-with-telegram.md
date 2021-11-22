---
layout: post
title:  "Server monitoring with Telegram"
permalink: /server-monitoring-with-telegram
date:   2019-05-19 11:18:00
tags: Server Monitoring Telegram Python
---

---

**_INFO:_** Those who already have a Telegram Botfather Token and want to skip the intro can directly access the [Github Repository](https://github.com/ahoz/Telegram-Server-Monitoring)

---

At the end of this intro, you will have logs like this:

![Telegram Monitoring Messages](/assets/images/telegram_message-1-1.jpg)

## Step 1: Create Telegram access token with Botfather

Open this link on your computer or phone: https://telegram.me/botfather and click on Send Message. You need telegram installed on your computer or phone. A new windows will open. Type in _/start_ and send the message. After that, some available commands will be shown.
Send _/newbot_ in order to crate a bot and set a unique name as requested by the bot. An access key will be shown to you if everything works out:

{% highlight text %}
> [...]Use this token to access the HTTP API: 123456789:AAEHUDJAN[...].
{% endhighlight %}

Now switch to the chat and write a message. A simple _Hi_ is sufficient. Now we need to find our the ChatID. This can be done by opening this url:


{% highlight text %}
> https://api.telegram.org/bot[ACCESS-TOKEN]/getUpdates
{% endhighlight %}

The output should like this:

{% highlight text %}
{"ok":true,"result":[{"update_id":123456789,
"message":{"message_id":3,"from":{"id":123456789,"is_bot":false,"first_name":"Ralph","language_code":"de"},"chat":{"id":987654321,"first_name":"Ralph","type":"private"},"date":1558263909,"text":"Hi"}}]}
{% endhighlight %}

The id that is shown here will be important in the next part.

## Step 2: Prepare server

Clone repository and setup


{% highlight text %}
ahmet@server1:~$ git clone https://github.com/ahoz/Telegram-Server-Monitoring
{% endhighlight %}
Adapt api key and chat id in monitoring.py. Insert your disk name in checkdisks and change the settings as you want.

## Step 3: Start monitoring

In order to monitor user logins the _/etc/profile_ file needs to be adapted.

{% highlight text %}
ahmet@server1:~$ echo "/opt/monitoring.py 1 >/dev/null 2>&1 ">> /etc/profile
{% endhighlight %}

Additional cronjobs are needed to monitor everything regulary

{% highlight text %}
ahmet@server1:~$ echo "0 */2   * * *   root    /bin/bash /opt/monitoring.py 2 >/dev/null 2>&1" >> /etc/crontab
ahmet@server1:~$ echo "0 */2   * * *   root    /bin/bash /opt/monitoring.py 3 >/dev/null 2>&1" >> /etc/crontab
{% endhighlight %}

Executing the scripts with user _root_ will always work. It is recommended to change this user to a non privileged user. I also want to emphasise that this script is developed in a short spare time. It should be polished before using it on a production environment.