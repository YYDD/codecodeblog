---
title: 科普URL
date: 2016-08-08 14:37:24
tags:
---
在一个机缘巧合的场景下，发现我对于url的组成好像有点略带懵逼。所以果断查了下资料科普了下。讲到url不得不说下uri。

### url和uri的区别

关于url和uri，对于我来说，之前都是懵逼的。URL的组成，也只是一知半解而已。今天好好的科普了下。顺带记录下~~
那么url和uri到底有什么区别呢？最简单的一句话就是url是uri的一个子集。官方的就是uri可以表示以为一个域也可以用来表示一个资源，而url只能用来表示一个资源。那么也就是说如果一串字符串是一个url那么它势必是一个uri。<!--more-->

##### uri（Universal Resource Identifier）通用资源标志符。通常由三部分组成：
- 访问资源的命名机制；
- 存放资源的主机名；
- 资源自身的名称，由路径表示。

##### url(Uniform Resource Locator)统一资源定位符。通常有三部分组成：
- 协议（服务器方式）
- 存放资源的主机IP地址（有时候也包括端口号）
- 主机资源的具体地址，如目录和文件名等


### url的组成

一个完整的url主要有protocol、hostname、port、path、parameter、query、fragment这六部分组成。下面是组成的顺序

```` objectivec
   protocol://hotsname[:port]/path/[paramerter][?query][#fragment]

```` 

##### protocol
访问协议（也称之为方案）：主要是告诉浏览器如何处理将要打开的文件，比较常见的有：http、https、ftp、mailto等协议。

##### hostname
hostname就是域名，有时候也可以由ip来代替

##### port
端口如果省略的情况下，将使用默认端口（80端口）

##### path
路径，网络资源在服务器中的指定路径

##### parameter
参数，向服务器传入的参数

##### query
查询字段，向服务器查询的内容

##### fragment
片段，如果访问网页中特定的内容位置时候需要设置该字段，就是锚点


### 收场
好了，科普文就到这里~对于url的知识，本来就没有多少。而且实际的运用中涉及到的个人感觉也没有很多。




