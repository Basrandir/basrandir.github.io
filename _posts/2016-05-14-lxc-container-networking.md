---
layout: post
title: Setting up LXC with Networking
category: guide
tags: how-to, lxc, networking
---
    ip link add name *bridge_name* type bridge
    ip link set *bridge_name* up
    ip link set *interface* master *bridge_name*
    dhcpcd *bridge_name*

    ping -c 1 www.bassamsaeed.ca

    lxc-create -n *lxc_name* -t debian

    /var/lib/lxc/*lxc_name*/config
    ## network
    lxc.network.type = veth
    lxc.network.link = *bridge_name*
