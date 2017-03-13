---
title: cat-client源码分析
date: 2016-10-10 10:25:23
tags: [dianping,cat,监控]
---


cat-client主要是业务程序使用，埋点生成各种消息，通过netty发送到cat server。

## 1. 初始化

在程序启动的时候，主要初始化了以下模块:

1. TcpSocketSender  用于连接cat服务器，通过netty异步发送消息(MessageTree)
2. ClientConfigManager 读取服务端配置，应用基本信息等(domain,appname,ip)
3. StatusUpdateTask 定时任务，每分钟获取一次心跳报文(HeartBeat)作为消息发送到服务器


## 2. 业务使用
在cat-client中，定义了3类消息(`Message`)

1. `Transaction` 事务类，可嵌套，可统计业务逻辑执行时间
2. `Event`	事件类，不可嵌套，近统计次数及衍生指标
3. `HeartBeat` 心跳报文，监控系统、JVM各种指标信息，与Message类似(该项cat已经配置好，一般不在业务代码中使用)

ps: cat的所有消息都是基于同线程收集，同一jvm不同线程的消息无关联

### 2.1 Transaction
使用事务消息，主要步骤是:

1. Cat.newTransaction(type, name) 
生成一个事务实例t，获取存储线程上下文信息，将该实例t加入MessageTree，加入堆栈（主要是为了判断嵌套）

	```
	DefaultMessageProducer.newTransaction()
		::m_manager.start(transaction, false)
			::ctx.start(transaction, forked)
	```

<!-- more -->


2. 执行业务逻辑
3. 加入附加信息，t.addData()  //该项可选
4. t.setStatus(true|false) //Message.SUCCESS = "0"表示成功，其他都表示失败
5. t.complete() 此项一定要执行，表示事务t的结束。     
	结束处理逻辑主要是:如果堆栈为空，则表示事务属于最外层事务，结束，将事务区间内的所有消息写入一个新的MessageTree，然后发送到本机的发送队列(netty从该队列异步获取数据发送)； 如果堆栈不为空，则表示该事务属于线程之前事务的一个子事务，则将事务压入堆栈，同时存储该事务到当前MessageTree(MessageTree中的Message对象可以增加无限的子Message)。

### 2.2 Event
### 2.3 HeartBeat
`Event`与`HeartBeat`的逻辑类似，只是各自承载的消息对象不同。他们与`Transaction`的区别在于前2者是不可嵌套的，所以不使用堆栈，在执行了complete()之后就获取MessageTree对象发送到带网络传输队列。

### 2.5 TCP Sender
所有消息在各自线程中的流向都是本地的MessageQueue, 其中有两条Queue
一条是`MessageQueue m_atomicTrees`原子消息队列,一条是`MessageQueue m_queue`发送队列。  其优化逻辑是:在将MessageTree加入发送队列`m_queue`之前，先判断是否是原子消息，如果是则加入`m_atomicTrees`，`m_atomicTrees`由一个独立的线程处理，如果队列中数量达到一定或者时间达到指定区间，则进行消息合并，以一定大小合并成多条Transaction对象，然后在加入待发送队列`m_queue`   

同样发送队列`m_queue`也是一个独立的线程处理，不断从队列中获取MessageTree,然后通过`PlainTextMessageCodec`序列化之后，通过netty发送到服务端。


至此，业务方的消息从收集到发送完成。
统计展示则需要通过服务端的分析处理后方可查看






