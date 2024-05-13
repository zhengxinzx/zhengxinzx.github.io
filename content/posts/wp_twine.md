---
title: "[White Paper Reading] Twine, cluster management system at Meta"
date: 2024-05-05
description: "A paper reading on the twine, the cluster management system in Meta."
tags: [White Paper Reading, Cluster Management]
---

Twine is the cluster management system at Meta, published at [Twine: A Unified Cluster Management System for Shared Infrastructure](https://www.usenix.org/conference/osdi20/presentation/tang).

Twine is a regional, managing the infrastructure of data centers consisting of millions of machines. It is designed to be power efficient and general for various types of workloads. To achieve this, there are 3 key design decisions they madej.

**Dynamic machine partitioning.** Instead of statically partitioning machines into different clusters and assign jobs to clusters, Twine design machines as dynamic resources, and can be added or removed from a logical unit called an *entitlement*. An entitlement is where th job is assigned to, thus the tasks of a job can be scheduled on machines across different availability zones (domains).This design significantly improves the utilization of machines, as idle machines can be reassigned to different entitlements.

**Customization in shared infrastructure.** Since machines are dynamic resources, there should be approaches to custom the machines to meet requirements of various workloads. Twine make this available by allowing their users to set up their own host profiles, so that when a machine joins an entitlement, before assigning tasks to it, Twine will configure the machine according to the host profile, for example, change the page size or switch the file system. Host customization avoid the need to specialize different machine types for different workloads, by simply enabling users to make the decision.

**Small machines.** As machines are dynamic and customizable, Twine further enhance the restriction on machines by using the same machine size: 1 18-core CPU with 64GB Ram. This design simplifies the management of machines, and also makes the machine more power efficient, they can make more fine-grained control at machine level.

In the paper, they mentioned Kubernetes multiple times as a comparison. Besides the one mentioned above, the key difference between Twine and Kubernetes are:

**Scalability.** While Kubernetes has a limitation of 5k nodes, Twine can scale to manage millions of machines.

**Sharding instead of federation.** The scalability of Twine is achieved by sharding the Twine, and each entitlement belongs to a shard. Honestly speaking, I personally think this is similar to Kubernetes federation, and the key difference is that Twine can scale more smooth by simply adding more shards.

**Fine-grained control by application feedback.** In Twine, there is a *TaskControl* API, where Twine send requests to applications to enable applications to decide which task can be deleted or rescheduled. This makes Twine more flexible, especially when managing stateful distributed systems such as a paxos-based key-value store: the kv store can decide which replica can be deleted to avoid downtime.

**Asynchronous rescheduling.** Once a pod is scheduled in Kubernetes, Kubernetes will not reschedule it unless it is deleted or the node is down. In Twine, the task is scheduled to a machine within a very short time to make the scheduling more efficient, thus it performs better when peak load comes. Then an asynchronous rescheduling is keep running, to reschedule tasks to improve the utilization of machines. This is a balanced design between performance and power efficiency.
