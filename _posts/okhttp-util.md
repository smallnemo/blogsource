---
title: okhttp-util
date: 2016-11-23 11:24:33
tags: [okhttp, httpclient]
---

# OkHttp简介
Github:[https://github.com/square/okhttp](https://github.com/square/okhttp)

OkHttp是square开源的一款高效Http请求组件。在Google Android4.4及以上系统，已经使用OkHttp替换掉了原有的HttpURLConnection。

主要有以下特点:

1. 支持HTTP2/SPDY协议
2. socket自动选择最好路线，支持自动重连
3. 拥有自动维护的Socket连接池(ConnectionPool),减少握手次数
4. 拥有队列线程池，支持异步并发请求
5. 拥有Interceptor逻辑(类似于Servlet的Filter插件)，可以方便快捷处理请求与响应数据(如压缩，日志等)
6. 拥有基于Header的缓存策略


# 支持范围
Android 2.3+
Java jdk1.7+

<!-- more -->


# HCB OkHttp组件
基于OkHttp做了封装，方便业务方使用。

在OkHttp功能基础上，针对业务及以往使用HTTP的要求，增加功能:

1. 签名、编码配置化
2. 一个目的业务服务一个client，互相不干扰
3.	结果反序列化
4.	请求构造多样化，API接口丰富
5.	调用简单化(方法仅有call， asyncCall; 参数仅有TARequest或者兼容以前的url)
6.	线程池复用(支持线程池空闲连接回收、已有连接健康监测)
7.	性能优化


主要组件:
	
	TAClient
	TARequest
	Serializer
	
## TAClient
http请求client，封装了具体的http连接客户端httpclient，主要配置参数是一些业务上的配置。主要包括以下参数：

	
	private HttpClient httpClient;

    //service url   //用来表示请求的目的服务
    private String serviceUrl;

    // TARequest 序列化 && Response反序列化
    private Serializer serializer;

    //appId
    private String appId;

    //签名密钥 私钥签名
    private String sign;

    //加密密钥 公钥加密
    private String enc;

    // 签名算法 RSA / MD5
    private CipherType signType;

    //加密算法 RSA / MD5
    private CipherType encType;
	

请求的每一个服务端可以配置一个TAClient，多线程复用。


在里面封装了HTTPClient对象，主要针对底层连接做一些配置，包括：

	//socket连接超时时间
	private long connectTimeout;

	//Default read timeout（ms）
	private long readTimeout;
	
	//Default write timeout (ms)
	private long writeTimeout;
	
	private List<Interceptor> interceptorList;

    // 线程池最大空闲连接数 默认100
    private int maxIdleConnections;
    //空闲保持时间:分钟 默认 5分钟
    private int keepAliveMinutes;

    //异步操作最大QPS
    private int asyncMaxRequests;

    //异步操作同一个目的Host:Port的最大并行数
    private int asyncMaxRequestsPerHost;

这些配置会生成一个可重用的无业务的Http连接客户端。所有的TAClient都可以引用同一个HttpClient.

TAClient和HtppClient对象都使用Builder Set模式，可以使用

	xx.set(xxx)
		.set(xxx)
		.set(xxx)
		

## TARequest

TARequest主要用于业务方每次生成自己的请求，最后作为HTTP调用的请求参数。

TAReqeust主要包括以下参数:

	//自定义url（一般不使用）
    private String url;
    
    //路径补充
    private String path;

    //是否编码
    private boolean isSign;
    // 是否加密
    private boolean isEnc;

    //request header 对应的keyList， valueList
    private final List<String> headerKey;
    private final List<String> headerValue;

    //request body对应的key，value
    /**
     * POST / GET
     */
    private String method;
    private final List<String> postBodyKey;
    private final List<String> postBodyValue;

    // request param 对应的keyList， valueList
    private final List<String> paramKey;
    private final List<String> paramValue;


根据URL的组成，分为几部分：
	
1. Header (keyList, valueList按位置一一对应)
2. Path (可添加path，不能修改serverUrl中配置url中的path，只能在后面添加)
3. QueryParam (url "?"后面的kv部分)
4. Method (Get 或者 POST， 默认GET)
5. PostBody （POST方法上传的KV数据，目前根据业务情况支持KV）
6. isEnc (如果为false，则确定不加密。其他情况根据TAClient是否配置秘钥来确定)
7. isSign (该次请求是否签名，如果false，确定不签名。否则根据TAClient是否配置签名私钥来确定)


## Serializer
在TAClient中我们有一个Serializer filed。主要作用是请求的序列化以及返回结果的反序列化，序列化使用统一格式，兼容公司现有http server。 结果反序列化默认使用fastjson进行序列化。一般线上返回结果都有Result<T>对象的JSON序列化字符串。

因此，如果在客户端要直接使用序列化后的结果，请使用

	taClient.call(taRequest, new TypeReference<Result<T>>(){})

如果返回结果不是容器泛型类对象，则可以使用

	taClient.call(taRequest, Class<T> clazz)



# 性能对比测试
我们主要和已有的组件进行对比。

|对比组件| hcb-okhttp | codec-http-client |
|------|------| ------|
|底层实现|httpclient-4.5 | okhttp-3.4.1|


## 实验环境
在本机创建一个HTTP Server，里面包括一个验证签名的filter
在servlet中代码实现如下:

```java
//计算所有参数的和，包括queryparam和requestbody content
Map<String,String[]> params = req.getParameterMap();
        int sum = 0;
        for (String [] values: params.values()) {
            if (values != null && values.length > 0) {
                try {
                    int value = Integer.parseInt(values[0]);
                    sum += value;
                } catch (NumberFormatException nfe) {

                }
            }
        }
        resp.getWriter().println(sum);
        
        //增加响应时间，使服务端响应时间更平稳
        long s = 0;
        for (int i =1; i< 10000; i++) {
            s += i;
        }
        double sqrt = Math.sqrt(s* s *s);
        
```	

## 实验结果

在客户端使用`3`个线程进行访问，输入参数如下:

```java
Map<String,String> params = new HashMap<String, String>();
params.put("a","1");
params.put("b","2");
params.put("c", "123");
params.put("d", "123");
params.put("e", "123");
params.put("f", "123");
```

对比结果如下:

|condition| 请求次数| codec-http-client响总应时间(ms)|平均响应时间(ms)| hcb-okhttp总响应时间(ms) | 平均响应时间(ms)|
|---------|------|-------------------|------------|-----------|-------|
|GET不签名 |1000_000| 80336| 0.080336| 52563| 0.052563|
|GET签名|300_000|29022|0.09674|23648|0.078827|
|POST不签名|1000_000|71709|0.07171|59762| 0.05976|
|POST签名|300_000| 244661|0.815537| 226051 | 0.7535|

从上面的测试结果可以看出，不论是否签名，使用GET还是POST方式，基于OkHttp的hcb-okhttp性能始终优于基于httpclient的codec-http-client。


# 快速使用

## Spring配置

pom文件中加入:

```
	<dependency>
		<groupId>com.wlqq.http</groupId>
    	<artifactId>hcb-okhttp</artifactId>
    	<version>1.0-SNAPSHOT</version>
	</denpendency>

```


Spring配置
  
```xml
	业务方需要使用的client
    <bean id="taclient" class="com.wlqq.http.TAClient">
        <property name="serviceUrl" value="http://10.2.1.3:5888/"/>
        <!--<property name="appId" value="xxxxxx"/>-->
        <!--<property name="cipherType" value="RSA"/>-->
        <!--<property name="sign" value="xxxxxx"/>-->
        <!--<property name="enc" value="xxxxxx"/>-->
        <!--property name="httpClient" ref="httpclient"/> -->
        
        <!--<property name="serializer">-->
            <!--<bean id="json" class="com.wlqq.http.serializer.JsonSerializer"/>-->
        <!--</property>-->
    </bean>
```

如果需要配置http连接池，超时时间等。需要增加:

```xml

	<bean id="httpclient" class="com.wlqq.http.HttpClient">
		 <!-- socket连接超时时间: ms，默认10_000ms-->
        <property name="connectTimeout" value="10000"/>
        <!-- readTimeout: ms，默认10_000ms-->
        <property name="readTimeout" value="10000"/>
        <!-- writeTimeout: ms，默认10_000ms-->
        <property name="writeTimeout" value="10000"/>
        <!-- socket连接保持时间:min分钟，默认为5分钟 -->
        <property name="keepAliveDuration" value="5"/>
        <!-- socket连接池最大空闲连接数，默认5个 -->
        <property name="maxIdleConnections" value="20"/>
        
        <!-- intercaptorList -->
        <!--<property name="interceptorList">-->
            <!--<list>-->
                <!--<ref bean="basicInterceptor"/>-->
            <!--</list>-->
        <!--</property>-->
        
        
    </bean>
    
    <!--<bean id="basicInterceptor" class="com.wlqq.http.interceptor.BasicInterceptor"/>-->

```


在taClient配置了一个请求服务，包括:

1. httpclient 默认的远程请求实现，可以不用配置，使用默认值
2. serviceUrl 远程服务的url地址，scheme + host+port+path，如: abc.com/get/user
3. cipherType 默认的签名、加密方式：RSA 或者MD5 //可以不写，默认RSA
4. appId	请求中用于编码需要的appId  
5. sign	签名private key //不写则默认不签名
6. enc		加密密钥	不填写则默认不加密
7. serializer 结果序列化转换工具，根据不同业务需求转换结果为需要的对象，默认为JsonSerializer, 将response body作为一个json反序列化为需要的Object


## JAVA使用

在java代码中调用，如果直接请求一个完整url,则直接使用 

	taClient.call(String url)

如果是复杂的request，则先创建TARequest

	TARequest taRequest = new TARequest()
	//request中添加数据
	
	
	String result = taClient.call(taRequest)
	
	//如果序列化非容器泛型
	T result = taClient.call(taRequest, Class<T> clazz)
	//如果序列化容器类泛型
	T result = taClient.call(taRequest, new TypeReference<T>(){})
	
	
	
	
	