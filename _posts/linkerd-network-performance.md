---
title: linkerd简介(8):network performance
date: 2016-08-24 15:58:10
tags: [linkerd,RPC]
---

原文链接: [https://linkerd.io/doc/network-performance/](https://linkerd.io/doc/network-performance/)

linkerd是一个网络代理，因此linkerd的性能问题主要就是隐藏在后面网络性能问题。为了帮助定位这些问题，下面是一些常见的网络性能问题查看方法。

## 1. LOOK AT BACKEND LATENCIES

linkerd会把流量路由到响应快的后端节点，但是如果pool太小或者所有的后端节点都是同样的延迟。ps:经常在测试环境中pool大小为1或者2，linkerd就无法使用负载均衡来优化性能。

查看`metrics.json` `loadbalancer`状态来查看pool的大小以及负载均衡的运行状态

在`metrics.json`中，其他的衡量指标是和后端节点的`connects`, `connections`，`request_latency`。

`/connects`：linkerd连接到一个pool的次数
`/connections`：到一个pool的开放连接数
`request_latency`：包含一个到pool的延迟直方图。这个与`loadbalancer/size`、`connections`一起能够帮组定位是否后端有足够的并发或者高效地处理当前的流量。

## 2. LOOK FOR DROPPED PACKETS
在一个拥堵的网络，丢包是有预示性的。网络拥堵将会导致99线，999线偶尔比预想值高。从linkerd的角度，因为网络拥堵引起的延迟与后端延迟是难以分辨的。

<!-- more -->

`ethtool -S eth0`

`/sbin/ifconfig eth0` 也可以报告丢包信息

一个症状就是999线将是200ms的N倍, 因为典型地操作系统超时重发的时间间隔是200ms


## 3.LOOK FOR DROPPED SYN PACKETS
linux有两个TCP的队列，一个是包数据准备好给应用程序，另一个是接受SYN，在TCP握手时的信号数据包。

如何查看丢失的SYN包:
`while true; do netstat -s |grep TCPBacklogDrop; sleep 5; done`
如果数据在不断增加，则是因为队列溢出而丢去SYN包。

## 4. LOOK AT SOCKET TYPES FOR HOSTS CONNECTING TO LINKERD
`watch -n 1 ss -tan 'sport = :4140'`   
查看处于`TIME_WAIT`状态的SOCKET连接，`TIME_WAIT`socket连接和丢失SYN包告诉我们连接池不在使用或者在请求很少的情况下，系统重用之前连接，到linkerd。

可以使用linkerd的`metric.json`来验证这些连接是否被重用或者在请求很少的时候被重用。



