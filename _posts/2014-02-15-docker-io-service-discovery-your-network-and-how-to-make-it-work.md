---
layout: post
title: "How I solved Service Discovery for Docker.io Using my existing DNS Server, Net Namespacing and Dynamic DNS"
description: "Docker.io service discovery, your own network and how to make it work: Solving Docker.io service discovery without additional docker containers or services"
category: devops
tags: [docker, paas, dns, bind, networking, scaling, service-discovery]
---
{% include JB/setup %}

__This post is about a script that listens to the [Docker.io][2] event cue: [that script can be found here][1].__

So about a year ago [Dotcloud][3] came out with a magical piece of software called [Docker.io][2]; a [Go-Lang][4] wrapper for [The Linux Container Engine (LXC)][5]. The first time I saw this I immediately jumped onto the [Docker.io][2] band wagon. Why? Because the idea is amazing! I had one problem with it though; the way [Docker.io][2] set's up your containers. You either have to setup your containers using their container linking solution, which puts all the container IP's into environment variables, or use a service discovery tool like [Skydock][6] with [Skydns][7]. Both of which are [Docker.io][2] containers themselves and required a pre-configuration process.

The second I saw [Skydock][6] and [Skydns][7] I thought it was great and really intuitive, but I already had a network I wanted to add the containers to. At the time I had been using the port forwarding feature provided by [Docker.io][2] and had exposed a ton of ports on the host OS, but this made it hard to work with services running on the same port. Especially when it came to service discovery and playing with multiple [Docker.io][2] hosts. There had to be a better way.

I was looking at [Pipework][8] and trying to hack the virtual ethernet interfaces to be configured by DHCP, but it felt like all my efforts were in vain and not working. So then I had an idea. What if it did not matter what the IP was? What if I could let [Docker.io][2] do it's thing and just add the virtual network to my existing network without the need for DHCP (for [Docker.io][2])?

This solution listens to the [Docker.io][2] events stream on the host OS and then updates your [Bind9 DNS][9] server with `nsupdate` and does a [Dynamic DNS][10] update to register your service. The best part is there are NO dependencies (aside from a network interface) for your guest OS or the way that you start your container. All this is possible thanks to [Linux Network Namespaces][13] which allow me to configure the virtual ethernet interfaces inside [Docker.io][2] containers from my host OS.

## How I did it

Before you try to implement my solution, this guide assumes you are using Bind9 DNS with [Docker.io][2] on an Ubuntu 12.04+ operating system. I already have a Bind9 DNS server with Dynamic DNS setup; for the purposes of this post I will be skipping over that.

If you alrady have Bind9 DNS installed with Dynamic DNS enabled: **Don't forget to ensure that [Dynamic DNS][10] updates are allowed from every [Docker.io][2] subnet!**

To start I would recommend setting up your own network bridge. You might be able to use [Docker's][2] but you probably will want to make some adjustments. I do this with the following code in my `/etc/network/interfaces` file.

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

To set this up there are a couple prerequisites. Just to be sure that I'm not getting any extra settings from [Docker.io][2] implicitly when it creates its bridge I usually delete it and re-create it manually (ensure the [Docker.io][2] service is not running).

    ifconfig docker0 down
    brctl delbr docker0
    brctl addbr docker0
    ifup docker0

If you used a configuration similar to my `/etc/network/interfaces` line then `ifup` will configure your bridge for you. If you restart your machine or run `service networking restart` it will also handle the above automatically for you.

After you have configured your bridge with custom subnets, DNS, etc... you need to update your [Docker.io][2] configuration to use your custom bridge (though it has the same name, I'm sourcing the variable just in case it changes how [Docker.io][2] internally handles network configuration; never hurts to be extra careful). To do this open up your `/etc/default/docker` configuration file, and add/alter the `DOCKER_OPTS` variable to contain `-b=docker0 -dns 10.1.0.1`.

Once this has been completed you can take the [`docker_ddns`][1] file and place it in `/usr/local/bin`. You need to make sure you either set the environment variables for [docker_ddns][1] or that you edit the script itself. The script is written in Ruby and the lines and the environment variables are:

    25: ENV['DDNS_KEY']   ||= "/etc/bind/ddns.key"
    26: ENV['NET_NS']     ||= "10.1.0.1"
    26: ENV['NET_DOMAIN'] ||= "kellybecker.me"
    28: ENV['DOCKER_PID'] ||= "/var/run/docker.pid"

- DDNS_KEY: Path to the key file used with nsupdate
- NET_NS: Address of the nameserver we will be updating
- NET_DOMAIN: Domain name to add docker containers to. (i.e. container.kellybecker.me; where 'container' is the [Docker.io][2] hostname '-h' option when using `docker run`)
- DOCKER_PID: Process ID file for Docker itself! (Used to determine Docker running status)

After you have that setup you should look at an upstart script or method of starting [docker_ddns][1] with [Docker.io][2]. I like to use [Monit][12] for service and process status monitoring. So here is my `docker_ddns.conf` file for [Monit][12].

    check process docker_ddns with pidfile /var/run/docker_ddns.pid
        start program = "/usr/local/bin/docker_ddns" with timeout 60 seconds
        stop program  = "/usr/bin/kill `cat /var/run/docker_ddns.pid`"
        if totalmem > 50.0 MB for 5 cycles then restart
        if 3 restarts within 5 cycles then timeout
        depends on docker
        group docker

I suppose you could also add the startup to your `/etc/init/docker.conf` though it won't ensure that the process does not die.

Once everything is all setup and running again; you can tail the logs of [`docker_ddns`][1] and you should see something along the lines of:

    I, [2014-02-18T09:32:55.978414 #1942]  INFO -- : Event Fired (8311c4c09153): create
    I, [2014-02-18T09:32:56.173839 #1942]  INFO -- : Event Fired (8311c4c09153): start
    I, [2014-02-18T09:32:56.429049 #1942]  INFO -- : Updated Docker DNS (8311c4c09153): container.kellybecker.me 60 A 10.1.0.2.
    I, [2014-02-18T09:37:23.389485 #1942]  INFO -- : Event Fired (8311c4c09153): die

## App Usage

    docker_ddns [ /path/to/log.file | - ]

    Using '-' will log output to the standard out. Defaults to '-'

[1]: https://gist.github.com/KellyLSB/4315a0323ed0fe1d79b6#file-docker_ddns "Docker DDNS, by Kelly Becker"
[2]: http://docker.io "Docker.io, by Dotcloud"
[3]: https://dotcloud.com "Dotcloud"
[4]: http://go-lang.org "Go-Lang"
[5]: http://linuxcontainers.org/ "LXC"
[6]: https://github.com/crosbymichael/skydock "Skydock"
[7]: https://github.com/crosbymichael/skydns "Skydns"
[8]: https://github.com.com/jpetazzo/pipework "Pipework"
[9]: https://www.isc.org/downloads/bind "Bind9 DNS"
[10]: https://www.erianna.com/nsupdate-dynamic-dns-updates-with-bind9 "Dynamic DNS with Bind9"
[11]: https://help.ubuntu.com/community/NetworkConnectionBridge "Ubuntu Network Bridge Guide"
[12]: http://mmonit.com/monit/ "Monit Process Monitoring"
[13]: http://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/ "Linux Network Namespaces"
