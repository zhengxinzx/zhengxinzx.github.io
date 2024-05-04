---
title: "Kube-proxy and CNI plugin: What's the difference?"
date: 2024-05-03
description: "Explain the difference between kube-proxy and CNI plugin, and describe how kube-proxy works briefly."
tags: [Networking, Kubernetes]
---

Understanding the components of a Kubernetes cluster is generally straightforward, as their names tend to hint at their functions, like *scheduler*. Nevertheless, the presence of kube-proxy alongside the CNI plugin within Kubernetes' networking architecture initially left me confused. My questions are:

- What are the differences between kube-proxy and the CNI plugin?
- Are they optional in a Kubernetes cluster?

To address these inquiries, it's important to delve into the of Kubernetes' networking model and how kube-proxy works.

![20240504174641](https://raw.githubusercontent.com/zhengxinzx/images/main/picgo20240504174641.png)

Kube-proxy is a process that runs on each worker node in the cluster. It can be deployed as a DaemonSet or a standalone process directly running on the host.

When a service object is created, an endpoints object is created automatically, which contains the IP addresses of the pods that the service is targeting. The service is assigned with a virtual IP address that does not belongs to any pod.

kube-proxy listens for the service and endpoints objects from the API server, and create network rules on the node, so that the packets sent to the virtual IP of the service will be routed to one of the pods in the endpoints object.

Depends on how the kube-proxy is configured, it can use iptables, ipvs, or nftables to create the network rules. The default mode is iptables nowadays, and you can find my previous post about iptables at [Iptables explore, implement virtual IP](https://zhengxin.online/posts/iptables/).

> The iptables mode is being challenged when there are large number of services in the cluster, since the iptables rules are evaluated in sequence, which can make the kernel quite busy. The ipvs mode is more efficient in this case, since it uses a hash table to find the correct pod to route the packet to.

On the other side, the CNI plugin is responsible for the pod-to-pod communication. It guarantees that each pod is assigned with a unique IP address, and pod can communicate with each other using the IP address without NAT, through technologies like VXLAN or BGP.

However, the functional boundaries between the two are becoming increasingly blurred, as the CNI plugins are trying to enhance their capabilities to provide the virtual IP support, like Cilium, as described in [Kubernetes Without kube-proxy](https://docs.cilium.io/en/stable/network/kubernetes/kubeproxy-free/#kubernetes-without-kube-proxy). Given that they furnish the cluster's network infrastructure, supporting virtual IPs is not a challenging task.

So the answer to the 2 questions are now straightforward:

- Kube-proxy and the CNI plugin serve different functions; however, the CNI plugin has the capability to replace the kube-proxy.
- Kube-proxy is not an indispensable component within a cluster. For instance, in Amazon EKS, it is considered an optional add-on. On the other hand, the use of a CNI plugin is essential, as you must select one to facilitate pod-to-pod communication within the cluster.

If you want to know more about kube-proxy, I'd like to recommend the following blogs:

- [Aks Networking Iptables in AKS](https://www.stevegriffith.nyc/posts/aks-networking-iptables/) explains how the iptables rules are configured in an AKS cluster.
- [Demystifying kube-proxy](https://mayankshah.dev/blog/demystifying-kube-proxy/) is a systematic introduction to kube-proxy.