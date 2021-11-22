---
layout: post
title:  "Setup your own Docker registry"
permalink: /setup-your-own-docker-registry
date:   2019-11-02 10:00:00
tags: Docker Server
---

Here is a short introduction on how to install your own Docker registry.

## Prerequisites

You need to have a working Docker environment and need basic knowledge of Docker containers and Dockerfiles.

## Setup

We are using the public [Docker registry image](https://hub.docker.com/_/registry).

{% highlight bash %}
ahmet@testserver1:~$ docker pull registry:latest
Using default tag: latest
latest: Pulling from library/registry
c87736221ed0: Pull complete
1cc8e0bb44df: Pull complete
54d33bcb37f5: Pull complete
e8afc091c171: Pull complete
b4541f6d3db6: Pull complete
Digest: sha256:8004747f1e8cd820a148fb7499d71a76d45ff66bac6a29129bfdbfdc0154d146
Status: Downloaded newer image for registry:latest
docker.io/library/registry:latest
{% endhighlight %}

Start the container:

{% highlight bash %}
ahmet@testserver1:~$ docker run -d -p 5000:5000 --restart always --name registry registry:latest
{% endhighlight %}

Check connectivity:

{% highlight bash %}
ahmet@testserver1:~$ curl -X GET http://localhost:5000/v2/_catalog
{"repositories":[]}
{% endhighlight %}

Since this is working fine, we can continue pushing our first image to our registry.

## Push & Pull

Lets pull a simple alpine image, retag it and push it to our registry:

{% highlight bash %}
ahmet@testserver1:~$ docker pull alpine:latest
latest: Pulling from library/alpine
89d9c30c1d48: Pull complete
Digest: sha256:c19173c5ada610a5989151111163d28a67368362762534d8a8121ce95cf2bd5a
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest
ahmet@testserver1:~$ 
ahmet@testserver1:~$ docker tag alpine:latest 127.0.0.1:5000/ymalpine:latest
{% endhighlight %}

Now let us check our catalogue:

{% highlight bash %}
ahmet@testserver1:~$ docker push 127.0.0.1:5000/ymalpine:latest
The push refers to repository [127.0.0.1:5000/ymalpine]
03901b4a2ea8: Pushed
latest: digest: sha256:acd3ca9941a85e8ed16515bfc5328e4e2f8c128caa72959a58a127b7801ee01f size: 528
ahmet@testserver1:~$ 
ahmet@testserver1:~$ curl -X GET http://localhost:5000/v2/_catalog
{"repositories":["ymalpine"]}
{% endhighlight %}

We are now able to pull our pushed image:

{% highlight bash %}
ahmet@testserver1:~$ docker rmi 127.0.0.1:5000/ymalpine
Untagged: 127.0.0.1:5000/ymalpine:latest
Untagged: 127.0.0.1:5000/ymalpine@sha256:acd3ca9941a85e8ed16515bfc5328e4e2f8c128caa72959a58a127b7801ee01f
ahmet@testserver1:~$ 
ahmet@testserver1:~$ docker pull 127.0.0.1:5000/ymalpine
Using default tag: latest
latest: Pulling from ymalpine
Digest: sha256:acd3ca9941a85e8ed16515bfc5328e4e2f8c128caa72959a58a127b7801ee01f
Status: Downloaded newer image for 127.0.0.1:5000/ymalpine:latest
127.0.0.1:5000/ymalpine:latest
{% endhighlight %}

## Conclusion

This is the shortest way to install a Docker registry. But there are still some steps missing.

1. Persistency: All pushed images will be gone after deleting the container. Think about using a docker volume.

{% highlight bash %}
docker run -d -p 5000:5000 --restart=always --name registry -v [DEIN PFAD]:/var/lib/registry registry:latest
{% endhighlight %}

1. Use a Docker registry ui: https://github.com/Joxit/docker-registry-ui

1. Install a reverse proxy with htpasswd and make it accessible via https. Otherwise it will only work locally and everyone could access your registry!

If you take those mentioned steps, your registry is production ready for small groups. If you are interested in enterprise or managed service, you can take a look at JFrog, Nexus, Gitlab and Github. These are only a few of many solutions.