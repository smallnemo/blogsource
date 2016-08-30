---
title: Finagle,Linkerd,and Mesos Twitter-style microservices at scale
date: 2016-08-25 15:50:17
tags: [finagle,linkerd,mesos]
---


<font color='red'>PPT FROM ApacheCon North America, May 2016</font>


## 1. Twitter
### 1.1 数据介绍 2010
10<sup>7</sup> users   用户   10<sup>7</sup> tweets/day 发推数     10<sup>2</sup> engineers  工程师   10<sup>1</sup> services   服务  10<sup>1</sup> deploys/week  发布/周   10<sup>2</sup> hosts    节点   0 datacenters   数据中心10<sup>1</sup> user-facing outages/week  人工介入的中断 

![Tweets per day](https://g.twimg.com/blog/s800/chart-tweets-per-day3.png)


### 1.2 Monorail 2010
10<sup>3</sup> of RPS   10<sup>2</sup> of RPS/host    
10<sup>1</sup> of RPS/process   

三层架构:

```
				hardware lb
				the monorail
			mysql	memcache	kestrel
```

主要问题:

1. Ruby perfomance 性能
2. Mysql scaling 扩展
3. MemCache operability  可操作性
4. deploys 发布

严重不对称，服务资源不足。

<!-- more -->

### 1.3 automating the datacenter
**mesos**: Abstracts compute resources   
`don’t worry about the hosts`

**aurora**: Schedules processes on Mesos 调度   
`no more puppet, monit, etc`

4层资源架构:

```
		appA		appB	 	appC
			Aurora(Marathon...)	
					mesos
host	host	host	host	host	host
```

### 1.4 microservices 微服务
**_flexibility_**:
	
	performance	高性能
	correctness	正确性	
	monitoring	可监控
	debugging		可调试
	efficiency	高效率
	security		安全性
	resilience	弹性

Resilience is an imperative: our software runs on the truly dismal computers we call datacenters. Besides being heinously  complex... they are unreliable and prone to  operator error.   
软件运行在普通的计算机组成的数据中心，除了架构复杂外，还是不可靠的，容易出现操作错误。

微服务中的resilience要求:
	
	software you didn't write
	hardware you can't touch
	network you can't configure

必须找到一种全新的方式，还要客户应用无感知。

![](/images/linkerd/7layer.png)


第5层在4层连接中分发请求。
就有了5层通信方案Finagle

### 1.5 	Finagle

github:[github.com/twitter/finagle](github.com/twitter/finagle)

主要特征:

	RPC library(JVM) RPC库
	asynchrounous 异步 
	built on Netty 基于Netty
	scala scala编写
		functional 函数式编程
		strongly typed 强类型
	first commit:2010.10
	
Finagle功能:
	
	transport security	安全传输
	service discovery	服务发现
	circuit breaking		中断
	backpressure			
	deadlines
	retries				重试
	tracing				路径追踪
	metrics				性能衡量
	keep-alive			长连接
	multiplexing			复用
	load balancing		负载均衡
	per-request routing	单一请求路由
	service-level objectives	
	

微服务框架中，服务组件越多，出现的问题也越多。


### 1.6 load balancing at layer 5负载均衡

算法:
	
	round-robin
	fewest-connections
	queue-depth
	exponentially-weighted moving average（ewma）
	aperture

![](/images/linkerd/load-balancing.png)



## 2. linkerd

github:[github.com/buoyantio/linkerd](github.com/buoyantio/linkerd)

特征:
	
	microserivce rpc proxy 微服务rpc代理(重点在代理)
		layer-5 router 5层上的路由
		l5d	简写为l5d
	built on finagle&netty
	pluggable	插件式
		http、thrift...
		etcd,consul,kuberbetes,marathon,zookeeper...
	

### 2.1 namerd
centralized routing policy
delegates logical names to service discovery
pluggable
	etcd
	kubernetes
	zookeeper
	...

![](/images/linkerd/namerd.png)


	
