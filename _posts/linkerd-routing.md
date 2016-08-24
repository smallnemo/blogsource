---
title: linkerd简介(3):routing
date: 2016-08-18 20:55:47
tags: [RPC,linkerd]
---

原文地址: [https://linkerd.io/doc/0.7.3/routing/](https://linkerd.io/doc/0.7.3/routing/)

linkerd的主要工作是路由:接收一个request，将request转发至正确的服务器。接下来将详细描述linkerd的如何做目的选择的。

这个过程包括4步：
	
	1. identification 
	2. delegation
	3. binding
	4. load balancing

该流程还有部分转移到`namerd`

![routing](https://linkerd.io/images/routing.png)

<!-- more -->

## 1. Identification

identification就是给请求分配一个名称(`name,path`)。 名称就是一个相当于请求目的地址的`画斜线的字符串`，linkerd默认使用`io.l5d.methodAndHost`，基于Http 方法和Host Header，如`/http/1.1/<METHOD>/<HOST>` 分配名称给请求。   
如:`GET http://example/hello` 将被分配一个名称 `/http/1.1/GET/example`
`/hello` 则在名称中被丢弃。但它任然会作为请求的一部分代理转发给服务端。name只是用来决定请求如何被路由转发，而不控制发送的内容。

<!-- more -->

**`identifer`**是一个插件模块，可以很容易的使用满足业务的自定义模块来代替。
更多关于identifer的配置，请看[linkerd identifer docs](https://linkerd.io/doc/0.7.3/linkerd/protocol-http#protocol-http-identifiers)      

identifer分配给请求的名称就是**`logic name`**, 它只会编码目的地地址，不会编码集群、区域环境、节点等信息。

如：应用想要请求'users'服务，会生成一个以`users`为Host Header的HTTP GET请求到linkerd。`io.l5d.methodAndHost`identifier将是`/http/1.1/GET/users`



## 2. Delegation
一旦一个**logic name**被分配给了一个请求，该name就接受dtab(`delegation table`)的信息。dtab工作原理见[dtab guide](https://linkerd.io/doc/dtabs). dtab编码**logic name**到**concrete name**转换的规则。一个**concrete name**就是一些列地址(服务发现入口)的集合。和**logic names**不同的是，**concrete name**包括集群、分区、环境等详细信息。

**concrete name**经常以`/$`,`/#`开头。

假设dtab信息如下：

```
/srv => /#/io.l5d.serversets/discovery
/host => /srv/prod
/http/1.1/* => /host
```
**logic name** `/http/1.1/GET/users` 将得到以下结果:

```
/http/1.1/GET/users
/host/users
/srv/prod/users
/#/io.l5d.serversets/discovery/prod/users
```
最后`/#/io.l5d.serversets/discovery/prod/users`就是**concrete name**	


## 3. Binding
**binding**就是讲**concrete name**映射到物理地址(ip+port)的行为。binding将由**namer**来完成。更多了解namer，见[linkerd namer docs](https://linkerd.io/doc/0.7.3/linkerd/namer)。



## 4. Load Balancing
linkerd有了服务的一组地址之后，使用[负载均衡算法](https://blog.buoyant.io/2016/03/16/beyond-round-robin-load-balancing-for-latency/)来决定请求发往的目的地。linkerd在请求层而不是连接层做负载均衡。负载均衡算法可以使用请求延迟信息来降权慢节点，避免更多负载过重的节点。

## 5. Namerd
**name interpretation**： 映射一个**logic name**到一堆地址。   
好处：   	

	* Decreased load on service discovery backends
	* Global routing policy
	* Dynamic routing policy
	