---
title: "[White Paper Reading] Conveyor: Continuous Deployment at Meta"
date: 2024-05-05
description: "A paper reading on the continuous deployment system at Meta, which is called Conveyor."
tags: [White Paper Reading, Continuous Deployment]
---

Meta has released a paper titled [Conveyor: One-Tool-Fits-All Continuous Software Deployment at Meta](https://www.usenix.org/system/files/osdi23-grubic.pdf), which offers an in-depth look at the company's state-of-the-art continuous deployment system, Conveyor. This system is integral to Meta's operations, deploying containers over an immense scale across millions of machines.

From the paper, three key insights emerge:

**Meta emphasizes the practice of continuous deployment without manual intervention.** The ingrained philosophy within the organization is to "release early, release often," with the aim of rapidly implementing bug fixes or security updates. Remarkably, 96.5% of regular services and 71.8% of large services at Meta are deployed automatically, enabling swift code evolution. This approach necessitated the development of Conveyor, a system that encapsulates Meta's unified approach to deployment. The successful implementation of full deployment automation at Meta signifies substantial commitment and collaboration between the infrastructure team responsible for Conveyor and the service teams who utilize it.

**It is not easy of executing in-place updates on a hyperscale site like Meta's.** Traditional blue/green deployment isn't practical due to the prohibitive resource requirements, such as doubling up on containers or virtual machines. To address this, Meta designed Conveyor to integrate with their cluster management system, Twine, and their monitoring system, Health Check Service (HCS). Conveyor deploys updates in incremental steps, refreshing only a subset of containers at a time and conducting health checks on these dynamic segments. This synergy between deployment, cluster management, and monitoring systems is what sets Conveyor apart from other deployment tools and is central to its functionality.

**The feedback loop from the service layer back to the infrastructure layer allows more nuanced control**. Services communicate with Conveyor through an interface called TaskControl, which specifies how many and which containers can be updated at any given time. This feedback mechanism is crucial, as it enables Conveyor to cater to a wide range of service requirements that it would not be able to manage independently. For instance, services like key-value stores, which depend on a distributed consensus algorithm, need to maintain a minimum number of operational instances to prevent outages. Only the service has the knowledge to determine which instances can be safely taken offline without compromising availability.
