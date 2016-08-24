---
title: linkerd简介(6):kubernetes
date: 2016-08-24 09:32:33
tags: [linkerd,RPC,kubernetes]
---


英文原文地址：[https://linkerd.io/doc/0.7.4/k8s/](https://linkerd.io/doc/0.7.4/k8s/)

本文主要介绍在Kubernetes里面部署linkerd。 关于Kubernetes的介绍，可以参考[Kubernetes初探](http://blog.csdn.net/zhangjun2915/article/details/40598151)

更多的关于部署拓扑的信息，请参考[Deployment Guide](https://linkerd.io/doc/0.7.4/deployment)。

首先，需要编写一个linkerd的配置文件。下面的例子配置了两个路由，一个在8080端口接收HTTP流量，并转发到一个监听7000端口的服务。另一个路由通过`disco`配置的服务路由内部流量。在此我们使用一个通过运行在8001端口`kubectl proxy`与K8s集群接口交互的k8s namer。

配置文件如下：

```
namers:
- kind: io.l5d.k8s
  experimental: true
  host: 127.0.0.1
  port: 8001

routers:
- protocol: http
  label: ext
  servers:
  - port: 8080
    ip: 0.0.0.0
  baseDtab: |
    /www          => /$/inet/127.1/7000;
    /http/1.1/*/* => /www;
- protocol: http
  label: int
  servers:
  - port: 4140
    ip: 0.0.0.0
  baseDtab: |
    /iface      => /#/io.l5d.k8s/$NAMESPACE;
    /srv        => /iface/rest;
    /http/1.1/* => /srv;
```

确保配置文件中的`Dtabs`能路由到想访问的所有服务。一旦配置文件确定之后，就必须把配置文件写入Kubernetes中运行的linkerd容器。推荐的方法就是使用[k8s secrets](http://kubernetes.io/docs/user-guide/secrets/)。

一个k8s scerets的例子：
<!-- more -->

```
---
kind: Secret
apiVersion: v1
metadata:
  name: linkerd-config
  namespace: prod
type: Opaque
data:
  config.yaml: $BASE_64_ENCODED_SECRET

```

使用卷挂载(volume mount)secret，将linkerd容器指向sceret提供的config.yaml文件。下面是使用上面secret的` Replication Controller object` 例子(部分)。

```
---
kind: ReplicationController
apiVersion: v1
metadata:
  name: web
  namespace: prod
spec:
  replicas: 1
  selector:
    app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      dnsPolicy: ClusterFirst
      volumes:
      - name: linkerd-config
        secret:
          secretName: "linkerd-config"
      containers:
      - name: service
        ....
      - name: l5d
        image: buoyantio/linkerd:0.6.0
        args:
        - "/io.buoyant/linkerd/config/config.yaml"
        ports:
        - name: router-ext
          containerPort: 8080
        - name: router-int
          containerPort: 4140
        - name: admin
          containerPort: 9990
        volumeMounts:
        - name: "linkerd-config"
          mountPath: "/io.buoyant/linkerd/config"
          readOnly: true
      - name: kubectl
        image: buoyantio/kubectl:1.2.3
        args:
        - "proxy"
        - "-p"
        - "8001"
```

最后，是一个针对上面RC Object的一个Service object例子：

```
---
kind: Service
apiVersion: v1
metadata:
  namespace: prod
  name: web
spec:
  selector:
    app: web
  type: LoadBalancer
  ports:
    - name: web
      port: 80
      targetPort: 8080
```

如果linkerd容器启动失败，并且返回如下错误信息：`"usage: linkerd path/to/config"`，就表示secret文件配置不正确。确保你的secret被正确挂载，并且挂载路径与linkerd容器输入参数匹配。