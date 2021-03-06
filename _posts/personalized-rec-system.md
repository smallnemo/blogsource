---
title: 个性化推荐系统简介 
date: 2016-08-15 13:35:46
tags: [rec,推荐系统,个性化]
---

## 1. 推荐系统背景
随着信息技术和互联网的发展，互联网的高速变化，人们逐渐从信息匮乏时代发展到信息过载的时代。 在信息海洋中，无论是信息的生产者还是信息的消费者，都遇到很大的挑战。一方是消费者要从海量信息中获取感兴趣的东西；另一方面信息生产者要从海量用户中找到潜在关注用户。  

信息获取的方式从：雅虎的分类导航、到搜索引擎(Google、baidu)、再到推荐系统(Amazon、Netflix)。

推荐系统正是解决海量生产者海量消费者模糊兴趣匹配的问题最佳选择。 推荐系统能在用户没有明确目的的情况下帮助用户发现他们感兴趣的新内容。  
  
推荐 VS 搜索

	搜索需要明确的用户需求
	推荐系统不需要明确的用户需求
	曾经的团购网站只有浏览，没有搜索框。

推荐系统使用场景：

	新闻阅读类（今日头条）
	电商网站(Amazon、Taobao、JD)
	社交网站(微博)
	音乐播放器(网易云音乐)
	个性化广告(精细化广告平台)
	基于位置服务的应用(大众点评、陌陌)
	

<!-- more -->
	

## 2. 推荐系统评价标准
如何判断一个推荐系统的好坏。推荐系统最终的目的是通过给用户提供其最感兴趣的物品，而提高转化率，提高用户粘性，最终促进整个生态圈活跃。

推荐系统连接：用户、平台、内容提供方 三者，达到共赢。

推荐系统主要有以下指标：

	用户满意度：用户是推荐系统的检验方，作为推荐系统的主要参与方，其满意度是评价推荐系统好坏的重要指标，主要通过用户行为的统计来计算，如点击率、停留时间、转化率等
	预测准确度：度量一个推荐系统或者推荐算法预测用户行为的能力。一般通过离线评测算法来验证。
	覆盖率：描述一个推荐系统对长尾物品发掘的能力。（推荐涵盖的物品占所有物品的比例）
	多样性: 推荐结果涵盖的范围，用户不同侧面兴趣的处理能力
	新颖性: 给用户推荐他们没看过但是可能会感兴趣物品的能力
	实时性: 不同的推荐系统对实时性的要求不同,如新闻类推荐对实时性要求就特别高

业务衡量指标：

	CTR（点击率）  点击数 / 浏览数 (请求数量）
	GMV（带来交易额）（客单价，每次请求带来的收益）
	
算法衡量指标:

	首次平均点击位置：平均值越小，越靠近1，则效果越好 （实际数据统计）
	平均点击次数：每次请求结果被点击的物品数量平均值
	平均点击位置：
	TopN: 推荐结果前N个结果被预测准确的比例 （离线模型评估） 
	RMSE：每个推荐结果得分与训练数据得分的均方根误差 (离线模型评估)
	FScore：准确率、召回率
	NDCG：推荐结果列表预测总分数评估（Normalized Discounted Cumulative Gain）不同的顺序位置会有不同的权重。 

 


| 数量 | 相关 | 不相关 |
|:------- | :------- | :------- |
|被检索到 | TP | FP |
|未被检索到 | FN | TN |
	
准确率：

	推荐结果中满足条件的结果占比
	P = TP / (TP + FP) （precision）
	
召回率：

	所有准确结果中，被找出来的数量占比
	R = TP / (TP + FN) （recall）

FScore:
	
	准确率和召回率的调和值。(F1值，准确率和召回率权重一样)
	2/F = 1/P + 1/R


	

## 3. 推荐系统数据
	物品数据（Item）
		车辆信息、货源信息
	用户数据（User）
		身高
		年龄
		职业
		性别
		注册城市
		常住城市
		兴趣偏好
		常出现位置
		最欢的分类
		注册时间
		活跃天数
		朋友关系
	行为数据（Action）
		查看物品行为
		点击行为
		购买行为
		停留时间
	当前环境数据
		POI
		当前城市
		当前接入网络
		当前App版本
		WIFI、4G
		天气	
	
## 4. 推荐系统引擎架构

![Netflix推荐系统架构](http://cdn3.infoqstatic.com/statics_s1_20160726-0420u1/resource/news/2013/04/netflix-ml-architecture/zh/resources/MachineLearningArchitecture-v3.jpg)

## 5. 推荐系统算法
### 5.1 社会化推荐
	向朋友咨询，利用朋友之间社会信任关系，从朋友那获取到的信息可能就是自己感兴趣的

### 5.2 基于内容的推荐
根据物品的属性、参数，推荐与该物品类似的item。
![基于内容的推荐](http://www.ibm.com/developerworks/cn/web/1103_zhaoct_recommstudy1/image007.jpg)
	
### 5.3 协同过滤
不考虑物品的属性，只参照用户和物品之间的行为。
	
Item-based: 基于用户关联的推荐

![item-based](http://www.ibm.com/developerworks/cn/web/1103_zhaoct_recommstudy1/image011.jpg)

User-based: 基于物品关联的推荐      
![User-based](http://www.ibm.com/developerworks/cn/web/1103_zhaoct_recommstudy1/image009.jpg)


相似度计算：

	余弦相似度：
		每个物品的属性都有一个向量表示。
		sim(a,b)=(a⃗ ⋅b⃗) /(|a⃗ |∗|b⃗ |)
	
	协同过滤物品相似度：
		N(i)表示喜欢物品i的用户数。
![](http://images.cnitblog.com/blog/554053/201309/24100430-0fdd4a3fb87e4ceda74d549fd97e1c51.png)	
### 5.4 机器学习的推荐
	L2R：Learn to rank, 将推荐问题转变为一个排序问题。
	分为
		PointWise
		Pairwise 
		Listwise
	将请求属性和结果物品的属性作为一个个的输入，根据算法模型，目标函数就是每个物品的分数、任意两个物品之间的先后顺序、所有物品的唯一先后顺序
	优化指标的不同，将待排序集合中的结果按照个体计算、任何两个作为一对来计算、所有结果一个列表来计算。
	
		
### 5.5 混合推荐
	在之前的各种推荐方法选择一种或多种进行第一次计算，将每一种算法的结果进行排列或者重新计算，得到一个新的推荐结果

## 6. 常见问题
	冷启动问题:
		新用户：实时信息的推荐、热门信息的推荐、利用用户注册时提供的信息及兴趣偏好选择进行推荐
		新物品: 基于内容的推荐，新品加权计算
	数据稀疏问题: 基于内容的推荐、利用用户注册信息等人口统计学特征
	
	

资源：
	L2R:[http://www.360doc.com/content/14/0218/16/13256259_353571486.shtml](http://www.360doc.com/content/14/0218/16/13256259_353571486.shtml)

