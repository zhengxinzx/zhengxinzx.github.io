---
title: "Kubernetes networking | Network Namespace, veth and bridge"
date: 2024-04-06
description: "This post will explain the fundamental concepts involved in Kubernetes networking, including network namespace, veth, and bridge."
tags: [Kubernetes, Networking]
---
Kubernetes guarantees that each pod in the cluster has a unique IP address, and they can can communicate with each other using this IP address directly. You may wonder how this is achieved. We will have a series of posts that will explain how networking works in Kubernetes. And this is the first post in the series, which will explain three fundamental concepts: network namespace, veth, and bridge, that are the building blocks of Kubernetes (and container) networking.

*In this post, we limit the discussion to the Linux networking, without considering the networking in other operating systems.*

## What are network namespaces?

Linux provides a feature called namespace. A namespace is a way to isolate resources in the system. For example, a process in one namespace cannot see the processes in another namespace. Linux provides several types of namespaces, such as PID namespace, mount namespace, network namespace, etc.

As described in [network_namespaces(7)](https://man7.org/linux/man-pages/man7/network_namespaces.7.html), 

> Network namespaces provide isolation of the system resources associated with networking: network devices, IPv4 and IPv6 protocol stacks, IP routing tables, firewall rules, the /proc/net directory (which is a symbolic link to /proc/pid/net), the /sys/class/net directory, various files under /proc/sys/net, port numbers (sockets), and so on.  In addition, network namespaces isolate the UNIX domain abstract socket namespace (see unix(7)).

There is one default network namespace in the system, where processes created without specifying a network namespace will be placed. 

We can create a network namespace using the `ip` command in `iproute2` package. For example, the following command creates 2 network namespace named `ns1` and `ns2`:

```bash
# Create a network namespace named ns1
$ sudo ip netns add ns1
# Create a network namespace named ns2
$ sudo ip netns add ns2
# List the network namespaces
$ sudo ip netns list
ns2
ns1
```

Thus there are 3 separate network namespaces in the system now, as shown in figure blow.

![20240406224442](https://raw.githubusercontent.com/zhengxinzx/images/main/picgo20240406224442.png)

This can be verified by checking the network interface devices in each network namespace:

```bash
# List the network interfaces in the default network namespace
$ sudo ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:47:1d:09 brd ff:ff:ff:ff:ff:ff

# List the network interfaces in the network namespace ns1
$ sudo ip netns exec ns1 ip link show
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

# List the network interfaces in the network namespace ns2
$ sudo ip netns exec ns2 ip link show
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

> The `ip netns exec` command is used to execute a command in a network namespace. In the above example, we use `ip netns exec ns1 ip link show` to list the network interfaces in the network namespace `ns1`.

As we can see, the network namespace `ns1` and `ns2` have only the loopback interface `lo`, while the default network namespace has both the loopback interface `lo` and the ethernet interface `eth0`.

## What are veth parirs?

## What is a bridge?
