---
title: linkerd简介(7):dtabs
date: 2016-08-24 14:16:11
tags: [linkerd,RPC]
---

原文地址: [https://linkerd.io/doc/dtabs/](https://linkerd.io/doc/dtabs/)

**`Dtabs`**就是一系列路由规则的集合，这些规则存储`logic path`(如: 一家货运公司)并且将他们转换为定位它们的`concrete name`(如:成都市天府软件园E3)。这个过程我们称之为`resolution`,它主要涉及一些列前缀的改写。

除了这些文件，你可以参考Twitter的[Finagle's dtab docs](http://twitter.github.io/finagle/guide/Names.html), 也可以通过运行linkerd实例后，浏览[http://localhost:9990/delegator](http://localhost:9990/delegator) 来体验dtab。


## 1. PATHS

最简单的dtab就是单条规则。(称为dentry)

	/56qqCorp => /softwarepark;

这条规则只对`56qq Corp`有效，对其他前缀的路径无效。

但是对于路径 `/56qqCorp/tech`, 前缀满足规则的左边部分。将会被替换为`/softwarepark/tech`。


## 2. DENTRIES & ORDER(次序)

**Dtabs**大部分时候有一些列dentry。 比如：

```
/smitten       => /USA/CA/SF/Octavia/432;
/iceCreamStore => /smitten;
/iceCreamStore => /humphrys;
```

当我们尝试解决匹配对个前缀的路径时，<font color='red'>**靠后的匹配优先**</font>。 所以路径`/iceCreamStore/try/allFlavors`首先会转换为`/humphrys/try/allFlavors`。然后，如果humphrys的地址无法感知，我们就会失败回退使用`/smitten/try/allFlavors`，进一步路径修改，最终变为`/USA/CA/SF/Octavia/432/try/allFlavors`。


### 2.1 STEP-BY-STEP EXAMPLE
比如dtab:
	
```
/iceCreamStore    => /smitten;
/smitten/try      => /smittenLocation/waitInLine/thenTry;
/smittenLocation  => /sanfrancisco/octavia/432;
/california       => /USA/CA;
/sanfrancisco     => /california/SF;
```
<!-- more -->

当给定路径为`/iceCreamStore/try/allFlavors`

解决步骤如下:

1. 首先匹配 `/iceCreamStore    => /smitten;`, 变为`/smitten/try/allFlavors`；
2. 匹配`smitten/try => /smittenLocation/waitInLine/thenTry;`，变为`/smittenLocation/waitInLine/thenTry/allFlavors`；
3. 匹配`/smittenLocation => /sanfrancisco/octavia/432;`，变为`/sanfrancisco/octavia/432/waitInLine/thenTry/allFlavors`；
4. 匹配`/sanfrancisco => /california/SF;`，变为`/california/SF/octavia/432/waitInLine/thenTry/allFlavors`；
5. 最后匹配`/california => /USA/CA`，变为`/USA/CA/SF/octavia/432/waitInLine/thenTry/allFlavors`。

从上面看到，每次都有一个前缀匹配，我们是匹配替换后的路径再次从底到顶搜索整个dtab。但是有时候我们会意外的制造路径替换循环，比如如下的dtab。

```
/iceCream        => /youScream;
/youScream       => /weAllScream/for;
/weAllScream/for => /iceCream;
```
不用担心，Finagle在多次循环的查找后会自动退出。


## 3. NAMERS & ADDRESS
到现在讨论的都还是路径的路由，为了使Finagle能成功路由一条请求，路径最终需要转换为一个`concrete name`。大部分的`concrete name`(在Finagle中叫做`bound addresses`)都是使用`namers`来定义和查找的。

Finagle提供了一个namer叫做`“/$/inet”`，它将定义随后的两个部分为ip和端口。所以"/$/inet/127.0.0.1/4140" 就被认为绑定地址`127.0.0.1:4140`

Linkerd也针对不同的服务发现机制提供了一系列namers。

一旦一个路径被namer转换为绑定地址(bound address)之后，路由就结束了，在前缀匹配中余下的部分任然不会被使用。

比如：一个namer定义了`/routeOnMethod`，它基于下一段为GET或者POST来转发请求，dtab就是`/http/1.1 => /routeOnMethod`，路径`/http/1.1/GET/host/users`将会被转换为`/routeOnMethod/GET/host/users`，前缀`/routeOnMethod/GET`将会被识别为一个绑定地址，剩下的`/host`和`/users`就对路由不起任何作用。


Namer不只是用来解决路径问题。 Namer的一个很基础功能是支持函数，如`/multiply`将会将后续的两个字段相乘返回一个数字。

比如dtab:

```
/byNine  => /multiply/9;
/byEight => /multiply/8;
/bySeven => /multiply/7;
```

路径`/byNine/3` 将返回`/27`。

## 4. ALTERNATES & UNIONS & WEIGHTS(候补、联合、权重)

当两个规则使用同样的前缀时，我们称之为可候补的。
如：

```
/smitten       => /USA/CA/SF/Octavia/432;
/iceCreamStore => /smitten;
/iceCreamStore => /humphrys;
```
可候补规则也可以使用管道符号"|"来改写：

```
/smitten       => /USA/CA/SF/Octavia/432;
/iceCreamStore => /humphrys | /smitten;
```

在两种情况下，`humphrys`就会优先被选择。如果一直失败回退到最后一个地址也是无法感知的，则该路径操作即使失败的。我们可以在可候补配置中增加任意个数的可选值。如`/humphrys | /smitten | /birite | /three-twins`...

Dtabs也支持联合，如下:`/iceCreamStore => /humphrys & /smitten`。在这个例子中任何一个地址都拥有相同的权重。如果我们更想到其中一个节点，可以增加它的权重(weights)。如:

```
/smitten       => 3 * /SF/Octavia/432 & 1 * /SF/California/2404;
/iceCreamStore => 0.7 * /humphrys & 0.3 * /smitten;
```
权重值可以是浮点数，也可以是整数。


## 5. NEGATIVE，FAILURE & EMPTY RESOLUTIONS

如果一个namer找不到一个`concrete name`,将会返回一个否定的方案。在给Finagle的信息中表明这个路径是不可用的，这时候可以使用一个可候补的路径去尝试。如果所有的路径最后都是错的，Finagle将抛出一个错误。这种错误重试逻辑可以使用`~`来测试。Finagle认为`~`是一个不可用的路径。
如:`/iceCreamStore => ~ | /smitten;`

如果我们想在检查所有候补路径之前停止，我们需要使用失败而不是否定。失败的描述是`/$/fail`或者更短的`!`。 如:
   
```
/iceCreamStore => /smitten | !;
```


Namers有时候会返回失败。如`/multiply`在路径`/multiply/cats/dogs`将可能返回失败。

一个在测试中用到的空方案是:`/$/nil` 或者`$`





