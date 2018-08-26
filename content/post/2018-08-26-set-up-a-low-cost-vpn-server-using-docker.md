---
title: Set up a low-cost VPN server using Docker
author: Steven Ge
date: '2018-08-26'
slug: set-up-a-low-cost-vpn-server-using-docker
categories:
  - cloud computing
tags:
  - Docker
header:
  caption: ''
  image: ''
---

[Amazon Lightsail](https://lightsail.aws.amazon.com/) now offers servers at $3.5 per month with 512MB of memory and 1Tb of data transfer. This make it possible to host personal VPN (virtual private network) servers cheaply. Docker containers enable quick configuration of these servers. 

1\. Create instance on Amazon Lightsail
---------------------------
Make sure you select the Linux Ubuntu instance without apps. Click on the instance, and then under Networking attach a static IP address to this instance. Firewall needs to be modified to allow VPN traffic through the 500 and 4500 ports. 
![ ](/img/vpn1.png)