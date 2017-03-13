---
title: cat快速入门
date: 2016-10-10 10:25:23
tags: [dianping,cat,监控]
---


cat-client主要是业务程序使用，埋点生成各种消息，通过netty发送到cat server。

快速使用方法:

## 1. dependency


	<dependency>
   		<groupId>com.dianping.cat</groupId>
       <artifactId>cat-client</artifactId>
       <version>1.3.7</version>    
    </dependency>   
    
## 2. 业务名称
在`main/resources/`下新建`META-INF`目录
在目录中新建文件 app.properties
里面写入: `app.name=businessName`  （这里写自己的业务名称）   


## 3. 埋点
CAT客户端可以向服务端发送Transaction, Event, Heartbeat三种消息
### 3.1 Transction
* transaction适合记录跨越系统边界的程序访问行为，比如远程调用，数据库调用，也适合执行时间较长的业务逻辑监控
* 某些运行期单元要花费一定时间完成工作, 内部需要其他处理逻辑协助, 我们定义为Transaction.
* Transaction可以嵌套(如http请求过程中嵌套了sql处理).
* 大部分的Transaction可能会失败, 因此需要一个结果状态码.
* 如果Transaction开始和结束之间没有其他消息产生, 那它就是Atomic Transaction(合并了起始标记).

<!-- more -->


代码示例:

```java
public void method() {
        Transaction t = Cat.newTransaction("Call", "test");  //type 和name 都可以自定义
        try {
            ///业务逻辑
            t.addData("xxx"); //可选项  附带数据，仅做展示用
            t.setStatus(Transaction.SUCCESS);  //status除了Transaction.SUCCESS，其他任何字符都表示error
        } catch (Exception e) {
            t.setStatus("error");
            Cat.logError(e);  //记录完整错误信息
        } finally {
            t.complete(); //该行一定要执行，所以一般使用try catch ，在finally中执行
        }
    }
```


### 3.2 Event

### 3.3 Heartbeat

心跳属于自动抓取，不需要业务代码埋点。 在展示的网页中点击`HeartBeat`即可查看。每台机器每个应用程序1分钟1次的数据


## 4. 查看
埋点完成之后，运行系统，消息数据将实时传输到服务器进行解析、分析和展示。
在网页上(http://10.2.1.3:5888) 可以看到统计信息.

* 找到业务名，就是在app.properties中声明的app.name
* 查看transaction和event等信息。页面显示的是埋点中`type`的统计信息，点击某一个`type`，可以看到对应的各个`name`的统计信息。
* problem中可以查看错误信息，点击后面`looooog`链接，可以查看所有的错误信息。
