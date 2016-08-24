---
title: linkerd简介(2):overview
date: 2016-08-17 17:03:31
tags: [RPC,linkerd]
---

原文链接：[https://linkerd.io/doc/0.7.3/overview/](https://linkerd.io/doc/0.7.3/overview/)

当前版本: <font color='green' size='+1'>0.7.3</font>


linkerd在JVM层面没有任何依赖，明确的设计能够适用于已有的应用程序。它能与很多的现有组件配合，如:RPC和服务发现机制设施、遥感指标、格式化规则、现代的展示如仪表盘、默认值、可控制范围的配置功能等。


## 1. 快速开始

linkerd以一个基础配合开始。该配置的路由规则是允许服务发现使用简单的[`file-based service discovery mechanism`](https://linkerd.io/doc/0.7.3/linkerd/namer#fs)

在这个简单配置中，linkerd将`loclahost:4140`的`HTTP`请求路由到`localhost:9999`, Host的HTTP Request header信息设置为‘web’。 同时会将thrift请求从`localhost:4141`路由到`localhost:9998`.

更多的使用linkerd在本地运行的资料见:[`Getting Started Guide`](https://linkerd.io/doc/getting-started/)

## 2. 如何使用

应用程序典型的使用linkerd方式：在已知位置部署运行linkerd实例，然后通过这些实例代理转发RPC请求。因为linkerd实例是无状态和独立的，所以可以独立地部署在各个环境中，最大限度地减少与应用程序的耦合。

通过延迟调用到linkerd，业务代码将与以下部分分离:

	1. 生产环境拓扑
	2. 服务发现机制设施
	3. 负载均衡和物理连接控制
	
应用程序也从该持续的、全局的流量控制设施中获得极大好处。这对与一个多种语言的程序特别重要。

## 3. 基本概念

下面有一些对理解linkerd十分有帮助的概念。

## 3.1 address
`address`就是一个ip加port的组合,一般通过`:`连接。如`localhost:1234`

## 3.2 concrete name

`concrete name` 就是一组地址的名称，典型地用于一些服务发现组件。`concrete name`可能会被存储到DNS或者Zookeeper. linkerd 仅仅把`concrete name`当做一个字符串，不会去解析它。

## 3.3 logic name

`logic name`就是一个RPC服务在运行时的描述。	如：一个名叫`users`的服务想请求`sessions`服务，logic name可能就是`sessions`。


# 4. 服务发现

大部分复杂的内部多个服务的程序都源于服务发现。但是，随着应用的复杂性增加和服务规模的扩大，服务发现变得越来越难以避免。linkerd通过以下方式来降低复杂度:
	
	1. 抽象出详细的底层服务发现机制
	2. 提供升级路径，能够选择合适的服务发现终点
	3. 在生产环境系统中多年的服务发现经验的基础上鼓励更好的实践
	
linkerd的服务发现机制是把服务发现信息当做一个可能的资源`advisory`，如果一个地址从服务发现中删除，linkerd可能会继续发送请求到改地址。相反地，如果一个地址被加入到服务发现中，linkerd也许会选举决定不发送请求到该地址(如load balance)。这源于多年的生产经验，它帮助保护服务发现中的失败。 


# 5. 负载均衡

对于所有可扩展的系统中，负载均衡都是一个十分关键的组件。通过智能分布流量到一个集合终端的各个节点，即使终端集合是动态更新的，当节点失败或者响应变慢的时候，一个优秀的负载均衡组件能够减少长尾延迟，提高系统的可用性。

linkerd提供了一系列优秀的负载均衡算法，包括 `least loaded`、`EWMA`、`aperture`。这些算法在Twitter等公司经过了大规模的测试验证。

