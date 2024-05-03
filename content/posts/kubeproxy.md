---
title: "Kube-proxy Explained with demo on Azure Kubernetes Service"
date: 2024-05-03
description: ""
tags: [Networking, Kubernetes]
---

When I first touched Kubernetes, I can understand that there are master components like API server, scheduler, etc. and worker components, which is kubelet. However, I was confused about kube-proxy and the concept of CNI plugin: Why do we still need kube-proxy when we already have CNI plugin? Should not the CNI plugin be responsible for the proxying network packet among pods?

To answer these questions, we need to explain what is kube-proxy and how it works.

## What is Kube-proxy

Kube-proxy is a process that runs on each worker node in the cluster. It can be hosted by a DaemonSet, or simply a process running on the node. Kube-proxy plays the role as a network rule configurator, which set up the network rule on the node, so that the network packet sent to/from the node can be routed to the correct pod. Based on these rules, Kubernetes provides the concept of service, which maps a virtual IP to a set of pods.

The difference between CNI plugin and kube-proxy is quite straightforward: CNI plugin solves the problem of how to build connection between pods, and kube-proxy solves the problem of which pod a network packet should be sent to.

> Kube-proxy is not strictly required for a kubernetes cluster. In Amazon EKS (Elastic Kubernetes Service), kube-proxy is an optional add-on. In fact, some CNI plugin are able to provide the same functionality as kube-proxy, such as Cilium.

## How it works

When a service, for example a ClusterIP service, is created in the kubernetes cluster, 

1. A virtual IP as assigned with the service. The virtual IP does not belong to any pod.
2. An endpoints object is created, which contains the IP addresses of the pods that the service is targeting.

On every worker node, kube-proxy listen for the changes/addition/deletion of endpoints object, and create network rules based on iptables/ipvs/nftables so that the network packet sent to the virtual IP will be sent to one of the IP addresses in the endpoints object. The virtual IP appears like a real IP address to the sender, where the packet was sent to a backend pod.

**The kube-proxy itself does not handle any network packet**.

Today, the default mode of kube-proxy is iptables. You can find my post about iptables at [Iptables explore, implement virtual IP](./iptables.md).

## Demo

Nothing can explain better than a real demo, and let's see how kube-proxy works on setting up the iptables rules.

In this demo, I will use an Azure Kubernetes Cluster (AKS) with two nodes, the cluster is configured to use the Azure CNI plugin.

### Preparation

## ClusterIP

## NodePort

## Setup

```shell
❯ kubectl get nodes
NAME                                STATUS   ROLES   AGE     VERSION
aks-agentpool-39551181-vmss000000   Ready    agent   4m19s   v1.28.5
aks-agentpool-39551181-vmss000001   Ready    agent   4m12s   v1.28.5

kubectl debug node/aks-agentpool-39551181-vmss000000 -it --image=ubuntu

apt install libcap2-bin iptables

setcap CAP_NET_RAW,CAP_NET_ADMIN+ep /usr/sbin/xtables-nft-multi

kubectl debug node/aks-agentpool-39551181-vmss000000 -it `
--image=ubuntu `
--overrides='{
  "apiVersion": "v1",
  "kind": "Pod",
  "spec": {
    "containers": [
      {
        "name": "debug-container",
        "image": "ubuntu",
        "securityContext": {
          "capabilities": {
            "add": ["NET_ADMIN"]
          }
        }
      }
    ]
  }
}'

```