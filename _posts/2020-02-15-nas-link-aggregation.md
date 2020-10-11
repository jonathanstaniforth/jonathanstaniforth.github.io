---
layout: post
title: "Increase Bandwidth to a NAS using Link Aggregation"
date: 2020-02-18 17:00:00 +0800
tags: [network]
navigation: true
---

When transferring large amounts of data between devices on a network the amount of bandwidth available can soon become a limiting factor.
Normally, to increase bandwidth you need to upgrade the network hardware in the devices, however, the increments in bandwidth are large, e.g. 100 Mb/s to 1000 Mb/s, and can be costly.
But, network cards can come with several Ethernet ports, allowing connections to multiple devices.

Link aggregation allows you to combine these multiple Ethernet ports into a single logical channel, enabling the bandwidth of each port to be utilised.
I recently used link aggregation to quadruple the available bandwidth to our Network Access Storage, allowing us to transfer significantly more data than previously possible.
Additionally, it provides failover for when an Ethernet port stops working.

# Link Aggregation
The idea of link aggregation is to distribute the packets of data between the ports so that the traffic is balanced.
Two stages are involved in setting this up: first, creating a link aggregation group; and, second, selecting a scheduling alogrithm.

## Link Aggregation Group
When a set of Ethernet ports are combined it is called a **link aggregation group**, with each port creating a separate link.

> All ports in the group must be identical, i.e. have the same bandwidth and configuration.

When sending data over the network, all the packets for a session must be sent over a single link.
This ensures that the packets are recieved in order.

## Scheduling Algorithm
Once the link aggregation group has been created, an appropriate scheduling alogorithm must be selected.
The key here is to use an algorithm that successfully randomises the packets across the links, resulting in the traffic being evenly balanced.

Selecting an appropriate algorithm depends on the situation in which link aggregation is being used.
When lots of different devices are communicating between each other, an algorithm that uses MAC addresses may be useful.
However, if the number of devices is small, a more appropriate algorithm may be based on IP addresses instead.

# Setup
Our NAS is a [Synology DiskStation DS1819+](https://www.synology.com/en-uk/products/DS1819+) which features four 1 Gb/s Ethernet ports.
This NAS makes it easy to set up link aggregation with the DiskStation Manager OS.

First, connect to the NAS using a browser and log in.

> Have Cat-5e/Cat-6 Ethernet cables ready to plug into the NAS.

Second, navigate to the network interface screen in the control panel:
**Control Panel > Network > Network Interface**.

Next, bring the Ethernet ports together by creating a bond:
**Create > Create Bond**.

Now, a screen will appear asking for the type of link aggregation mode to enable.
Which mode to select depends on the switch that the NAS is connected to.

The switch that we are using is a [TP-Link TL-SG1016D](https://www.tp-link.com/us/business-networking/unmanaged-switch/tl-sg1016d/), which has 16 ports at 1 Gb/s.
Unfortunately, this switch does not support link aggregation protocols, however, this is not a problem as we can still select the **Adaptive Load Balancing** option.
This allows for the incoming and outgoing traffic to be divided between the Ethernet ports, ultimately reducing the congestion found in the network.

Once the link aggregation mode has been set, select the Ethernet ports that you want grouped. In my case, I selected all 4 available ports.

Next, set up the network as required. If you have a DHCP server connected, then you can select DHCP.

Finally, click apply.
At this point, plug in the Ethernet cables into the NAS and then into the switch.

> Make sure to plug the Ethernet cables into the ports of the NAS that were selected previously.

Once you have plugged in the cables, check the network interface to make sure the interfaces have been detected and the network status is correct.
As you can see below, I have grouped the 4 Ethernet ports to achieve a 4 Gb/s bandwidth.

![Synology Network Interface](/assets/images/synology-network-interface.jpeg)

