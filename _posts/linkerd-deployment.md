---
title: linkerd简介(5):部署
date: 2016-08-23 16:42:44
tags: [linkerd, RPC]
---

原文地址:[https://linkerd.io/doc/0.7.3/deployment/](https://linkerd.io/doc/0.7.3/deployment/)

除了JVM,linkerd没有其它依赖。它很明确的设计来适应已存在应用的部署，可以和多种RPC协同工作。linkerd主要有两种部署方式：
	
1. **per-host**
2. **be a sidecar process** 

## 1. PER-HOST部署
![](https://linkerd.io/images/diagram-per-host-deployment.png)

在per-host部署模式中，每个host(物理还是虚拟)部署一个linkerd实例，在改主机的所有应用服务实例都使用同一个linkerd实例进行RPC路由。

这种模式主要适用于host-based部署。主机上的每一个服务应用实例能在固定位置(如：`localhost:4140`)找到对应的linkerd实例，而不需要客户端额外的配置。

这种模式要求linkerd实例支持高并发，需要较多的资源支持。这种模式，一个独立的linkerd实例相应地失去了主机本身。

<!--more-->

## 2. SIDECAR部署
![](https://linkerd.io/images/diagram-sidecar-deployment.png)

在sidecar部署模式中，每一个业务服务对应的部署一个linkerd实例。这种模式主要适用于主要实例、基于容器的服务实例等与host-based完全相反的实例。如在一个Kubernetes部署中，一个linkerd容器可以被当做Kubernetes “pod”的一部分来部署。服务实例可以访问linkerd实例，就像是在同一个host。

sidecar部署需要很多的linkerd实例，每个需要较少的资源。这种模式，独立的linkerd实例相应地失去了与服务本身的映射。



------------------
业务应用服务于linkerd交互有三种配置：

1. **service-to-linker**
2. **linker-to-service**
3. **linker-to-linker**

### 2.1 SIDECAR SERVICE-TO-LINKER部署
每个服务实例通过对应的linkerd实例路由RPC请求。每个linkerd将服务对应的在固定位置的实例，并且将流量路由到远程服务。

### 2.2 SIDECAR LINKER-TO-SERVICE部署
应用服务实例不直接接受流量。相应地，sidecar linkerd在服务发现中被注册，因此直接接受进入的流量，并路由到对应的服务实例。

### 2.3 SIDECAR LINKER-TO-LINKER部署
以上两种配置的合并，能够表现出两者的好处。linkerd在服务发现中注册，因此流量进入首先经过linkerd，然后转发到对应的服务实例，服务实例在运行调用返回结果时，通过linkerd返回结果。这几需要在linkerd配置中有两个路由:incoming 和outgoing


一个涵盖三种模式的配置项：

```
namers:
- kind: io.l5d.fs
  rootDir: disco

routers:

# Use this router for linkerd-to-service
# This server should be registered in service discovery so that incoming traffic
# is served here.
- protocol: http
  label: incoming
  servers:
  - port: 4140
    ip: 0.0.0.0
  # Route all incoming traffic to the application service
  # (assumed to be running on port 8080)
  baseDtab: |
    /local => /$/inet/127.1/8080 ;
    /http/1.1/*/* => /local ;

# Use this router for service-to-linkerd
# The matching service instance should send all outgoing traffic to port 4141
# so that linkerd may route it appropriately.
- protocol: http
  label: outgoing
  servers:
  - port: 4141
    ip: 0.0.0.0
  # Route outgoing traffic based on the Host header
  baseDtab: |
    /srv      => /io.l5d.fs;
    /http/1.1/* => /srv;

# Use both above routers for linkerd-to-linkerd
```