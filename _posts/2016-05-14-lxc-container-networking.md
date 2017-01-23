---
layout: post
title: Setting up LXC with Networking
category: guide
tags: how-to, lxc, networking
---
From my understaning Ubuntu will setup a bridge by default when you install lxc. Or perhaps when you first create a container. I'm not certain. This isn't the case in a lot of other distributions however.

In this guide we'll setup a network bridge. The bridge with be setup on the host system to aggregate the internal container networks with the outer network. In this way the host system will become a bridge from the router to the containers.

First create the virtual link and bring it up.

    ip link add name *bridge_name* type bridge
    ip link set *bridge_name* up
    
Add an interface (presumably the LAN interface) to the bridge.

    ip link set *interface* master *bridge_name*

Get an IP address for the bridge.

    dhcpcd *bridge_name*

Confirm that the host has a proper connection.

    ping -c 1 www.bassamsaeed.ca

If everything is working then the bridge is working perfectly. The next step is to properly set up the containers to use the bridge.

First create the container. Use any preferred template.

    lxc-create -n *lxc_name* -t debian

Edit the config file of the container. This is usually located at `/var/lib/lxc/*lxc_name*/config`. Add the following two lines:

    ## network
    lxc.network.type = veth
    lxc.network.link = *bridge_name*

This will create an virtual ethernet interface and link it with the bridge.

The guest container is now bridged with the network. DHCP may be configured in the guest container. Static IP may also be configured in the actual container or in the container config file as follows:

    lxc.network.ipv4 = 192.168.1.100/24
    lxc.network.ipv4.gateway = 192.168.1.1
