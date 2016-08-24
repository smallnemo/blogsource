---
title: GitHub+Hexo+Godaddy快速搭建个人博客
date: 2016-08-17 10:16:48
tags: [Github,Hexo,Godaddy]
---



# 1. 背景

最近在折腾博客，然后室友就推荐用github，免费。而且有很多很方便的工具，他推荐了hexo。

这两天也在自己看着教程搞，中途难免有一些坑。现在将总体流程及一些小的Tips记下来。
最后再加上了自己的域名，就是现在的这个网页了。

言归正传，主要步骤如下(适用于Mac和Linux)：

	NodeJs+Hexo (就这两个有先后顺序)
	Github
	Godaddy
	
# 2. 主要步骤

ps:<font color='red'>事先提醒一下，很多时候发现安装出错，第一解决方法就是命令前加上sudo，用管理员权限安装，这招屡试不爽。</font>

<!-- more -->

## 2.1 Node

首先Hexo需要NodeJs支持，这个容易，使用apt-get(linux)或者homebrew之类
的软件管理工具都ok。
我选择直接去官网下载安装了，这步基本没什么注意事项，都可以搞定。
安装完成之后，使用
> node -v
查看是否安装成功。

## 2.2 Hexo

Hexo的安装也属于傻瓜式的。还是记住一点，一般错误都先加sudo来尝试解决

主要命令有:

>安装hexo module  
>npm install hexo -g

>//进入你准备存储博客的文件夹，初始化一个博客(比如我的博客文件夹是smallnemo)   
>$ hexo init smallnemo  
>
// generate   
$ hexo g （generate的简化命令）
>   
//server 本地启动   
$ hexo s （server简化命令）  
 
这时候Hexo其实都算是在本地搭建起来了。可以在浏览器中输入 localhost:4000 查看博客内容了
Hexo暂时告一段落，之后还会有一点点配置。


## 2.3 Github

这一段会比较的复杂了，但是对于熟悉git或者用过github的人来说也是比较容易的。

使用github，首先你的有一个账号，需要用一个邮箱去注册的
(比如 邮箱是testmail@163.com，账号是testaccount)
注册完成之后，需要一个ssh key的添加，以及repository的创建。

下面的步骤需要仔细看了:

	在github新建一个repository，如果用户名是testaccount,
    则repository名字必须为 testaccount.github.io

接下来就是ssh-key了,如果注册邮箱是testmail@163.com，则
>$ ssh-keygen -t rsa -C "testmail@163.com"    

在这条命令执行时要求输入内容: 如果你本机没有使用其他邮箱执行过上面的命令，则直接回车，回车再回车就好。反之如果你本机已经使用过其他邮箱执行上面的命令，则需要重新定义文件名,如~/.ssh/github_id_rsa,第一次要求输入时写入前面的文件名，之后不管，只要回车就好。
然后查看公钥信息，vim ~/.ssh/github_id_rsa.pub(如果之前没有输入文件名，则是打开id_rsa.pub),将文件里所有信息复制，在github网页中去加入ssh-key。
打开https://github.com/settings/profile， 左侧导航找到 SSH keys， 
然后Add SSH Key，
title里随便写，在key输入栏粘贴刚刚的公约信息。 这样ssh-key就搞定了。

## 2.4 Hexo + Github

这一步主要是配置博客内容推送到github。
在博客文件夹下有一个`_config.yml`文件，打开文件直接跳到最后，看到deploy选项。

```
deploy
    type: 
```

在这儿增加内容，修改为(记住`冒号后面一定得加上一个空格`, repository后面的内容
可以在github上面去复制，根据自己的账号名称去修改):

```
deploy
    type: git
    repository: git@github.com:testaccount/testaccount.github.io.git
    branch: master
```

## 2.5 Hexo 推送

经过上面的配置，博客只差2条命令就ok了。

>$ hexo g (generate简写)    
>$ hexo d

如果出现权限错误，请加sudo
如果第二条命令执行无效(无输出无报错)，请再加上一条命令
>$ npm install hexo-deployer-git --save    

然后再去
>hexo g  
>hexo d

就搞定了

最后就可以去浏览器访问 testaccount.github.io了。

## 2.6 Godaddy

觉得这个域名不够漂亮，想使用自己的域名，先去godaddy申请了一个域名。

先在博客文件夹中新建一个文件CNAME，在里面输入申请的域名。
然后
>hexo g   
>hexo d

剩下的主要就是dns配置了。

在DNS配置中：   
增加A(Host)：

```
Host    Points To
@       192.30.252.153
@       192.30.252.154
```
然后再增加一个CName(Alias)

```
Host    Points To
www     testaccount.github.io
```

保存，这下testaccount.github.io就可以通过在godady上申请的域名访问了。
至此，Hexo + Github + Godaddy 配套就搞定了。然后就可以通过 `Hexo new “newBlogName”`新建博客文章了。
