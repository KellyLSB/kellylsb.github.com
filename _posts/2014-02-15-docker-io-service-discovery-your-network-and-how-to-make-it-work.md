---
layout: post
title: "Docker.io service discovery, your own network and how to make it work"
description: "Solving Docker.io service discovery without additional docker containers or services"
category: devops
tags: [docker, paas, dns, bind, networking, scaling, service-discovery]
---
{% include JB/setup %}

This post is about the the code available at: [https://gist.github.com/KellyLSB/4315a0323ed0fe1d79b6#file-docker_ddns](https://gist.github.com/KellyLSB/4315a0323ed0fe1d79b6#file-docker_ddns)

So about a year ago dotcloud came out with a magical piece of software called Docker.io; a Go-lang wrapper to The Linux Container Engine (LXC). The first time I saw this I immediately jumped onto the Docker band wagon. Why, because the idea is amazing. I had one problem with it though, due to the way Docker set's up your containers. Uou either have to setup your containers using their container linking solution which puts all the container ip's into environment variables or use a service discovery tool like Skydock with Skydns. Both of whihch are Docker containers themselves and required a pre-configuration process.

The second I saw Skydock and Skydns I thought it was great and really intuitive, but I already had a network I wanted to add the containers to. For the time being I had been using the port forarding tool provided by Docker and had exposed a ton of ports on the host OS, but this made it hard to work with services running on the same port. Especially as it came to service discovery and playing with multiple Docker hosts. There had to be a better way.

I was looking at Pipework and trying to hack the virtual ethernet interfaces to be configured by DHCP, but it felt like all my efforts were in vain and not working. So then I had an idea. What if it did not matter what the IP was. What if I could let Docker do it's thing and just add the virtual network to my existing network without the need for DHCP (for Docker).

This solution listens to the Docker events stream on the host os and then pings your Bind9 DNS server with nsupdate and does a dyanmic dns update to register your service. Best part is there are NO dependnecies (aside from a network interface) for your guest os or the way that you start it.

So I created a network bridge using Ubuntu's bridge-utils package and went to work. Here is what I came up with.

## How I did it

This guide assumes you are using Bind9 DNS with Docker on an Ubuntu 12.04+ operating system.

To start I would recomend setting up your own network bridge. You might be able to use Docker's but you probably will want to make some adjustments. I do this with the following code in my `/etc/network/interfaces` file.

```
auto docker0
iface docker0 inet static
    bridge_ports none
    bridge_fd 0
    address 10.1.0.1
    netmask 255.255.0.0
    network 10.1.0.0
    broadcast 10.1.255.255
    gateway 10.1.0.1
    dns-nameservers 10.1.0.1
    dns-search <search-domain>
    post-up route add default gw <external_interface> || /bin/true
```

Though to set this up there are a couple prerequistes. Just to be sure that I'm not getting any extra settings from Docker implicitly when it creates it's bridge I usually delete it and re-create it manually (ensure the Docker service is not running).

    ifconfig docker0 down
    brctl delbr docker0
    brctl addbr docker0
    ifup docker0

If you used a configuration similar to my `/etc/network/interfaces` line then ifup will configure your bridge for you. If you restart your machine or run `service networking restart` it will also handle the above automatically for you.

After you have configured your bridge with custom subnets, dns, etc... you need to update your Docker configuration to use your custom bridge (though it has the same name, I'm sourcing the variable just in case it changes how docker internally handles network configuration; never hurts to be extra careful). To do this open up your `/etc/default/docker` configuration file. And add/alter the `DOCKER_OPTS` variable to contain `-b=docker0 -dns 10.1.0.1`.

Once this has been completed you can take the `docker_ddns` file and place it in `/usr/local/bin`. I like to use Monit for service and process status monitoring. So here is my `docker_ddns.conf` file for Monit.

```
check process docker_ddns with pidfile /var/run/docker_ddns.pid
  start program = "/usr/local/bin/docker_ddns" with timeout 60 seconds
  stop program  = "/usr/bin/kill `cat /var/run/docker_ddns.pid`"
  if totalmem > 50.0 MB for 5 cycles then restart
  if 3 restarts within 5 cycles then timeout
  depends on docker
  group docker
```

I suppose you could also add the startup to your `/etc/init/docker.conf` though it won't ensure that the process does not die.

## App Usage

    docker_ddns [ /path/to/log.file | - ]

    Using '-' will log output to the standard out. Defaults to '-'
