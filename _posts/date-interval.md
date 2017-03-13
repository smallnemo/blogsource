---
title: 求两个日期之间相隔的天数
date: 2017-02-13 16:13:50
tags:
---

# 背景
在项目中遇到一个需求，计算一个证件还有多少天到期，在还有30天到期的时候给一个提醒。看起来很简单，就是求给定的时间和当前时间的一个差值。

题目可以理解为：
给定一个日期(Date)，计算与当前时间相差的天数。

# 第一次
这个很简单嘛，先找找JAVA有木有对应的API，Date只有before，after方法，返回boolean值，不可取，再查查看Calendar也没有对应的方法。

好嘛，没有我就自己写，几行代码的事情。

```
public int intervalDays(Date date) {
    if (date == null) {
        return 0;  //这儿忽略，根据业务场景定义返回值
    }
    long oneDayMillis = 24 * 60 * 60 * 1000;  //一天的毫秒数
    return (int) ((System.currentTimeMillis() - date.getTime()) / oneDayMillis);
}
```
其实一行代码就可以搞定，简单嘛。   

接下来发现不对了，计算的是天数差距，如果当前是`12点`，给定日期是`昨天的15点`，上面代码计算出来的时间不足1天，返回的该是`0`。 而我们理解及业务要求中，这种情况应该返回`1`的。

<!-- more -->


# 第二次
好嘛，要这样搞，我又用天数来相减嘛。

	当前时间的天数 减去给定时间的天数
	
代码更新如下:

```
public int intervalDays(Date date) {
    if (date == null) {
        return 0;  //这儿忽略，根据业务场景定义返回值
    }
    long oneDayMillis = 24 * 60 * 60 * 1000;  //一天的毫秒数
//    return (int) ((System.currentTimeMillis() - date.getTime()) / oneDayMillis);
    //总天数如果算上当天 两个天数都要+1， 在这里计算间隔，就都不算当天
    return (int) (System.currentTimeMillis() / oneDayMillis - date.getTime() / oneDayMillis); 
    }
```
这下肯定没问题了，我用天数相减，万无一失。

可是现实是很残酷的。     
比如我现在的时间是 `2017-02-13 07:00:00`    
给定的日期是 `2017-02-12 19:00:00`
一眼看出来，相差是一天，因为只看天数是有差距的。

代码运算出来的时间却是 `0`。    
赤裸裸的打脸呀...


# 终章
马上想到一个问题，System.currentTimeMillis()取的是UTC的时间，我们用的是CST时间，求差值没问题，但是我们在求差值之前取了整数呀。    

于是乎找了一个时间来验证

	$ date -r 1486915200   根据秒数查看日期
	Mon Feb 13 00:00:00 CST 2017

于是乎:
   
    >>> 1486915200.0 / (24 * 60 * 60)
    17209.666666666668  查看天数，返回的居然不是整数，居然是这种小数，晕了。
仔细一看，小数好像是`2/3`，还差 `1/3`为整数，正好8小时(东八区)是一天的 `1/3`。

于是乎，在东八区，所有时间计算天数需要在UTC时间上加上 8小时算出来的天数才是准确的。

于是又自己造轮子:

```
public int intervalDays(Date date) {
    if (date == null) {
        return 0;  //这儿忽略，根据业务场景定义返回值
    }
    long oneDayMillis = 24 * 60 * 60 * 1000;  //一天的毫秒数
//    return (int) ((System.currentTimeMillis() - date.getTime()) / oneDayMillis);
    //总天数如果算上当天 两个天数都要+1， 在这里计算间隔，就都不算当天
//    return (int) (System.currentTimeMillis() / oneDayMillis - date.getTime() / oneDayMillis);
        
    long addMillis = 8 * 60 * 60 * 1000;
    return (int) ((System.currentTimeMillis() + addMillis) / oneDayMillis 
            - (date.getTime() + addMillis) / oneDayMillis);
}
```

再次验证，这次好像可以了吧。

# 总结
在JAVA中，所有时间相关的计算，都要随时想到时区这个东西，一不小心就会发现，这是一个很容易掉下去的坑。
 


