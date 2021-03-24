---
layout: posts
title: Exploring Internetworking in Linux (Layer2/Layer3 survey)
categories: Linux_Networking 
author: "Keyvan"
published: true
---

# Exploring Internetworking in Linux (Layer2/Layer3 survey)
In this post, we are going to explore Linux commands, which helps us to look around the layer 2 functionality in the Linux networking stack.
## Bridging
If you are using Open Switches and open network Linux OS (Like cumulus OS), or have a bridge on your Linux machine then you need check the MAC address table.
Bridge (traditionally been dedicated hardware devices) can connect two or more network interfaces together. 
You can add two interfaces to a Linux bridge with **ip link set** and **ip link add** using:

    keyvan@iproute:~$ sudo ip link add bridge1 type bridge
    keyvan@iproute:~$ sudo ip link set eth1 master bridge1
    keyvan@iproute:~$ sudo ip link set eth2 master bridge1

in this example **ip link add** command, is creating a bridge named bridge1  and **ip link set** commands add the two Ethernet interfaces, eth0 and eth1, to the new bridge as result we have a connection between these two interfaces.

lets see the content of the **MAC address table** using **bridge** command.

    keyvan@iproute:~$ sudo bridge fdb show
    33:33:00:00:00:01 dev eth0 self permanent
    01:00:5e:00:00:01 dev eth0 self permanent
    01:80:c2:00:00:0e dev eth0 self permanent
    01:80:c2:00:00:03 dev eth0 self permanent
    01:80:c2:00:00:00 dev eth0 self permanent
    33:33:ff:8d:c0:4d dev eth0 self permanent
    08:00:27:91:16:05 dev eth1 master bridge1 permanent
    08:00:27:91:16:05 dev eth1 vlan 1 master bridge1 permanent
    33:33:00:00:00:01 dev eth1 self permanent
    01:00:5e:00:00:01 dev eth1 self permanent

**fdb** stands for *forwarding database management*.

Also you can use **bridge-utils package**, it contains a utility needed to create and manage bridge devices. 

    keyvan@iproute:~$ sudo brctl showmacs bridge1
    sudo: unable to resolve host iproute
    port no	mac addr		is local?	ageing timer
      2	08:00:27:57:51:c0	yes		   0.00
      2	08:00:27:57:51:c0	yes		   0.00
      1	08:00:27:91:16:05	yes		   0.00
      1	08:00:27:91:16:05	yes		   0.00

## Spanning Tree
Spanning Tree Protocol (STP) is a Layer 2 protocol that runs on bridges and switches. The specification for STP is IEEE 802.1D. The main purpose of STP is to ensure that you do not create loops when you have redundant paths in your network. Loops are deadly to a network.
if you accidentally create loops, then you will ultimately bring the network down. 
we can mitigate these loops through the use of spanning trees.
A spanning tree is always recommended for any bridge or device configured with a bridge interface to prevent bridging loops, reduce broadcast traffic, and provide automatic failover if you have redundant links.
You can perform most spanning tree configurations in Linux by using the **mstpd-ctl** command, which controls the *multiple spanning tree protocol daemon* (MSTPD).
