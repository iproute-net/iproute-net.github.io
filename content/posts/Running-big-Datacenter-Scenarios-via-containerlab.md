+++
title = "Running big Datacenter Scenarios via containerlab"
description = "Running Datacenter scenarion using container lab"
date = 2023-08-11T09:19:42+00:00
updated = 2023-08-11T09:19:42+00:00
draft = false
template = "blog/page.html"

[taxonomies]
authors = ["Keyvan Ghadimi"]
+++

In this article I want to share some tips I faced in using clab([_Containerlab_](https://containerlab.dev)).
I am using **CentOS Stream release 8** in an old server with Gen1 intel xeon (Intel(R) Xeon(R) CPU E5-2640 0 @ 2.50GHz) and 128GB of RAM.
I used Arista cEOS 4.27.3F. 
some of the scenarios are explained here: [_Arista Validated design with cEOS-lab_](https://github.com/arista-netdevops-community/avd-cEOS-Lab) 

## Tip#1:
In versions prior to EOS-4.28.0F, the ceos-lab image requires a cgroups v1 environment.
You need to add "systemd.unified_cgroup_hierarchy=0" to your kernel parameters. This is achieved by editing /etc/default/grub line like this:

    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash systemd.unified_cgroup_hierarchy=0"

then run 'sudo update-grub' and reboot.


## Tip#2:
If you have a big Scenario you may faced some issue in the performance and your system crashed with some errors like: 
**"coredump datagram"** , **"inotify cannot be used, reverting to polling: Too many open files"**
These problems also related to some default configuration in Arista cEOS.
I have identified a problem in cEOS, when it starts, it modifies some parameters (see file /etc/sysconfig/99-eos.conf of the image) and in particular /proc/sys/fs/inotify/max_user_instances to 1256 I solved the problem by modifying this file on the host and by adding a "binds" example : 

    topology: 
      nodes: ceos: 
        kind: ceos 
        image: ceos:4.26.3M 
        binds: - /myhomelabspath/my-99-eos.conf:/etc/sysconfig/99-eos.conf	

In my local file (my-99-eos.conf), I changed: fs.inotify.max_user_instances=from 1256 to 65536

    fs.inotify.max_user_instances=65536

## Tip#3:
if you have too many images to run simultaneously  you may have concern about the resources you can set [_bootdely_](https://containerlab.dev/manual/vrnetlab/#boot-delay)
also if you are using a linux and wants to have lacp you have to added this one to **env**

	nodes:
	  server1:
	    kind: linux
	    mgmt-ipv4: 172.30.255.29
	    mgmt-ipv6: 1001:172:30:255::29
	    env:
	      TMODE: lacp
	      BOOT_DELAY: 300


## Tip#4:
if your scenario is really huge and have maybe hundreds of switches you can cluster somehow the containerlab.
for this purpose you can use [_Multi-node labs_](https://containerlab.dev/manual/multi-node)
you may follow the examples in the above link, but some more tips is here to help you.

    topology:
      kinds:
        ceos:
          image: ceos:4.27.3F
      nodes:
        sw02:
          kind: ceos
          mgmt_ipv4: 172.210.100.102
          mgmt_ipv6: 1001:172:210:100::102
          binds:
            - ./99-ceos.conf:/etc/sysctl.d/99-eos_k1.conf
        br-e1:
          kind: bridge
        br-e2:
          kind: bridge
    
      links:
        - endpoints: ["sw02:eth1", "br-e1:e1"]
        - endpoints: ["sw02:eth2", "br-e2:e2"]

now you need to follow this steps:

 1. create some bridges in your linux machine 
 2. connect the physical interface to a switch 
 3. allow required vlan on the port (if it is trunk). in below example I used vlan 11 and 12
do the same steps in the second machine.
### creating bridges in Linux:

    ip link add br-e2 type bridge
    ip link add br-e1 type bridge
    ip link set up br-e1
    ip link set up br-e2
    ip link add link eno1 name eno1.11 type vlan id 11
    ip link add link eno1 name eno1.12 type vlan id 12
    ip link set dev eno1.11 master br-e1
    ip link set dev eno1.12 master br-e2
    ip link set up eno1.11
    ip link set up eno1.12

