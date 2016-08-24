---
title: linkerd简介(4):快速开始
date: 2016-08-23 15:30:23
tags: [linkerd, RPC]
---


原文地址：[https://linkerd.io/doc/getting-started/](https://linkerd.io/doc/getting-started/)    

简单使用linkerd的基本步骤。

## 1. 需要组件
运行linkerd，需要java8
> $ java -version   
> java version "1.8.0_66"

linkerd支持Oracle和OpenJDK的JAVA。

[Download Oracal Java8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)	
[Download OpenJDK8](http://openjdk.java.net/install/)


## 2. 下载和安装
下载linkerd: https://github.com/BuoyantIO/linkerd/releases

下载之后：
>$ tar -zxf linkerd-0.7.3.tgz   
>$ cd linkerd-0.7.3

<!-- more -->

文件夹包含以下文件:
	
1. `config/linkerd.yaml` 定义路由、服务、协议和端口等的配置文件
2. `disco/` 基于文件的服务发现配置
3. `docs/` 文档
4. `linkerd-0.7.3-exec` 可执行文件
5. `logs`  	默认日志文件夹

## 3. 运行linkerd

解压之后，就可以运行linkerd了.
> $ ./linkerd-0.7.3-exec config.linkerd.yaml


## 4. 测试验证

使用发送HTTP请求验证linkerd是否work。linkerd被配置为监听`4140`端口, 路由任何设置Host header为“web”的HTTP请求到一个监听`9999`端口的服务。

运行一个监听9999端口的服务来测试：
>$ echo 'hello world' > index.html
>$ python -m SimpleHTTPServer 9999
这样在本地启动了一个9999端口的http服务。

当我们使用http请求4140端口的时候:
>$ curl -H "Host: web" http://localhost:4140/
>hello world

当我们使用Host不是web的4140端口来测试:
>$ curl -H "Host: foobar" http://localhost:4140/
>HTTP/1.1 502 Bad Gateway

## 5. 发现服务
启动linkerd后，当需要查找一个服务节点的时候，第一个查看的是`disco`文件夹。

默认的配置是：

	$ head disco/*
   	==> disco/thrift-buffered <==
   	127.0.0.1 9997

   	==> disco/thrift-framed <==
   	127.0.0.1 9998

  	==> disco/web <==
  	127.0.0.1 9999
  	
从上可以看到一个“web”的目的地址使用了:`127.0.0.1 9999`。 thrift framed使用了`127.0.0.1 9998`, thrift buffer使用了`127.0.0.1 9997`

该文件配置了所有的服务发现节点，所有linkerd能通过坚持改文件夹的变化，可以在不重启的情况下增加、删除、编辑任何一个节点的配置。

更多的关于路由的配置，见[linkerd's routing capabilities](https://linkerd.io/doc/0.7.3/overview/#routing)

## 6. 管理
linkerd在9990端口运行了一个web管理界面。启动linkerd之后可以通过 http://localhost:9990 来访问。
使用管理界面可以用来查看linkerd的操作状态，修改一些运行中的参数。

## 7. 衡量指标（Metrics）
Linkerd发布机器可读的指标数据。通过一个额外的指标集合工具，这些指标数据可以被用来计数。 运行linkerd之后，可以通过http://localhost:9990/admin/metrics.json 来查看。