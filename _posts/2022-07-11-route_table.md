---
title: How to access two nets in the same time?
description: what is network route table?
categories:
 - tutorial
tags: [learning]
---

# 缘从何起之如何通过两个网卡同时访问内外网？
# How to communicate between networks?
every router creates a boundary between two networks, and their main role is to forward packets from network to the next.

| Target Networks | Next-Hop/Interface |
| --------- | ---------------------- |
| 11.11.11.x | eth0 |
| 22.22.22.x | eth1 |
| 33.33.33.x | 22.22.22.2(与该路由器在同一网络内的下一跳的路由器ip地址) |


routing table

| Target Networks | Next-Hop/Interface |
| --- | --- |
| 11.11.11.x | eth0 |
| 22.22.22.x | eth1 |
| 33.33.33.x | 22.22.22.2 |


netstat -nr
route -n
ip route

对一般在ubuntu上查到的路由表而言，如果网关为 0.0.0.0 则表示该网络直连在该网络设备上，目的地址为 0.0.0.0 的路由则表示的是默认网关，因为目的地址可以匹配任何ip。

# how to add routing entry?

Reference
https://www.practicalnetworking.net/series/packet-traveling/host-to-host-through-a-router/
http://t.zoukankan.com/lexiaofei-p-13114574.html
