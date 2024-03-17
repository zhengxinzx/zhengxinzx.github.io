---
title: "What is DNS and how it works in Kubernetes"
date: 2024-03-17
description: "This post describe how DNS works, with real examples of nslookup, and explain the details of DNS resolution within Kubernetes."
tags: [Kubernetes, Networking, DNS]
---
## What is DNS

DNS, which is short for **domain name system** was designed aim to solve a simple but critical problem in computer networking: how to translate human-friendly domain names into IP addresses.

## How DNS works

DNS is a distributed database, implemented in a hierarchy of name servers. There are 3 types of DNS servers: 

- **Root servers**: These servers are the first step in the DNS lookup process. They are responsible for answering requests for the top-level domains (TLDs) such as .com, .org, .net, etc. There are 13 (logical) root servers in the world, managed by 12 different organizations.
- **TLD servers**: These servers are responsible for answering requests for the second-level domains (SLDs) such as google.com, facebook.com, etc.
- **Authoritative servers**: These servers are responsible for answering requests for the specific domain names such as www.google.com, mail.google.com, etc.

When a client wants to resolve a domain name, for example the www.google.com domain, it will
  
- First, send a request to the root server to ask for the IP address of the .com TLD server. The root server will respond with the IP address of the .com TLD server. The response could contain a list of IP addresses, and the client will choose one of them to send the next request.
- Then, the client sends a request to the .com TLD server to ask for the IP address of the google.com authoritative server. The .com TLD server will respond with the IP address of the google.com authoritative server, which also could contain a list of IP addresses.
- Finally, the client sends a request to the google.com authoritative server to ask for the IP address of the www.google.com domain. The google.com authoritative server will respond with the IP address of the www.google.com domain.

