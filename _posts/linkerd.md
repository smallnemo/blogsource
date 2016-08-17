---
title: linkerd调研<1>(Introduction)
date: 2016-08-16 14:47:22
tags: RPC
---


## 1. Intro

原文链接:[https://linkerd.io/doc/introduction/](https://linkerd.io/doc/introduction/) 

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

<center>![](https://linkerd.io/images/diagram-individual-instance.png)    
<font size='+1'>linkerd作为应用程序的一个RPC代理 </font></center>

通过从应用程序代码中解耦RPC的机制，linkerd允许在不修改业务代码的基础上改变RPC机制。如：linkerd通过升级路径来实现服务发现， 以简单的静态主机列表，达到zookeeper或者consul等的全面解决方案。

通过提供一套持续通用的仪表展示和服务控制，独立的服务程序语言， linkerd让用户选择任何适合他们服务的语言。

linkerd搭建在twitter的大规模RPC库Finagle基础之上。在Twitter还是其他公司，Fingale能够很好的完成复杂应用和高流量的各种业务调用。通过使用Finagle，linkerd可以利用经过千万级RPC调用流量测试验证过的代码、算法和方法。





### 1.1 linkerd's feature

* Intelligent, adaptive load-balancing. 智能的、自适应的负载均衡。 因为操作在RPC层，linkerd可以根据`RPC调用延迟`和`队列长度`来进行负载均衡，这比诸如LRU或者TCP活跃性的启发式机制更好。 通过linkerd可以优化流量分布和减少长尾延迟。
* Global, fine-grained instrumentation. 全局的、细维度的仪表数据展示。linkerd提供通信延迟、负载量的详细矩阵图数据，也包括成功率和负载均衡的统计，以人识别的展示方式和机器识别的展示方式。使用linkerd，即使是多种语言的应用，也可以在集中一个地方查看到一个连续的、全局的性能数据。
* Application-centric naming 以应用为中心命名
* Powerful traffic routing mechanisms.路由机制。 linkerd的路由层解耦了应用的流量服务拓扑和部署拓扑。 这意味着：灰度部署、ABTest部署、暗流量等，可以在流量控制中快速、控制范围、可回滚的实现，而不是代码部署。


### 1.2 linkerd使用

linked作为一个分离的独立的代理运行，与语言和需要的库无关。相比于与目的服务应用直接通信，本地的linkerd实例可以看成是远程的目的服务实例，应用程序只用连接到本地的linkerd实例。在这里，linkerd接受路由规则、与存在的服务发现设施通信、目的实例的负载均衡、所有的通信和报告指标。

linkerd实例有两种部署方式：

	1. sidecars  一个应用服务实例对应一个linkerd实例
	2. per-host	一个目的host一个linkerd实例
	
linkerd实例是无状态和独立的，可以快速加入已存在的部署拓扑中。

linkerd支持多种通用的RPC格式。如：<font color='red'>`HTTP、mux、thrift`</font>



