---
title: linkerd
date: 2016-08-16 14:47:22
tags: RPC
---

# Linkerd调研

## 1. Intro

原文链接:[https://linkerd.io/doc/introduction/](https://linkerd.io/doc/introduction/)  Introduction to linkerd

linkerd是一个专门为微服务设计的RPC代理， 它帮助服务实现：

* safe 安全
* reliable 可靠
* performance 高性能
* load balancing 负载均衡
* connection pooling 连接池
* session management 会话管理
* instrumentation 仪表盘数据
* routing 路由

linkerd用来解决类似于Yahoo、Twitter、Google和MicroSoft等公司的大型业务系统问题。在我们的经验中，最复杂的问题并不是服务本身，而是服务之间的通信。linkerd就是用来解决这些问题的：不仅控制通信机制，而且在上层提供一层抽象。



