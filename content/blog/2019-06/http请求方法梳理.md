---
date: "2019-06-19T17:19:48+08:00"
draft: false
title: "http请求方法梳理"
tags: ["网络协议", "计算机网络", "http"]
series: ["计算机网络"]
categories: ["计算机网络"]
toc: true
---

## 序
我们都知道HTTP的报文结构，它是由header+body构成，请求头里有请求方法和请求目标，响应头里有状态码和原因短语，
今天要说的就是请求头里的请求方法。
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/http_01.png" width='600px'/>{{% /center %}}


## 标准请求方法
HTTP协议里的"请求方法"是什么呢？    
请求方法就是客户端发出一个"动作指令"，要求服务器端对URI定位的资源执行这个动作。    
目前HTTP/1.1规定了八种方法（单词必须大写）：

>`GET：获取资源，可以理解为读取或下载数据；`    
>`HEAD：获取资源的元信息（不下载）；`    
>`POST：像资源提交数据，相当于写入或上传数据；`    
>`PUT：类似POST；`    
>DELETE：删除资源；    
>CONNECT：建立特殊的连接隧道；    
>OPTIONS：列出可对资源实行的方法；    
>TRACE：追踪请求 - 响应的传输路径。     

### GET/HEAD
`GET`方法是请求服务器获取资源，这个资源可以是静态的文本、图片、视频、页面，也可以是由PHP、
JAVA等动态生产的页面或其他格式的数据。    
GET方法还可以搭配URI和其他头字段就能实行对资源更精细的操作。     
如：`URI后面加上"#"，就可以做描点用`；
使用If-Modified-Since字段就变成了“有条件的请求”，仅当资源被修改时才会执行获取动作；
使用Range字段就是“范围请求”，只获取资源的一部分数据。

`HEAD`方法可以看做是GET方法的简化版。它的请求头与GET完全相同，但他并不会下载资源。    
当你要坚持一个文件是否存在，只要发个HEAD请求就可以了，没有必要用GET把整个文件都取下来。

### POST/PUT
`POST/PUT与GET/HEAD是相反操作。向URI指定的提交数据，数据就放在body里。`

PUT的作用与POST类似，但与POST存在微妙的不同。
通常，`POST表示的是“新建”“create”的含义，而PUT则是“修改”“update”的含义。`

### DELETE
`DELETE`方法指示服务器删除资源，因为这个动作危险性太大，所以服务器通常不会执行真正的删除操作，
而是对资源做一个删除标记。更多时候服务器直接就不处理DELETE请求。

### CONNECT
`CONNECT`是一个比较特殊的方法。要求服务器为客户端和另一台远程服务器建立一条特殊的连接隧道，
这时web服务器在中间充当了代理的角色。

### OPTIONS
`OPTIONS`方法要求服务器列出可对资源实行的操作方法，在响应头的Allow字段里返回。
它的功能有限，用户也不大，有的服务器（如nginx）干脆就没有实现对它的支持。

### TRACE
`TRACE`方法用于对http链路的测试或诊断，可以显示出请求-响应的传输路径。它的本意是好的，
但是存在漏洞，会泄露网站的信息。所以通常服务器也禁止使用。

## 扩展方法
虽然HTTP/1.1里面规定了八种请求方法，但是并未限制我们只能用这八种方法。我们可以添加任意请求动作，
只要请求方和响应方都能理解就行。

主要得到实际应用的请求方法（WebDAV）：MKCOL、COPY、MOVE、LOCK、UNLOCK、PATH等。

常用的有：使用LOCK方法锁定资源暂时不允许修改，或者使用PATCH方法给资源打个小补丁，部分更新数据。
                          
## 安全与幂等
关于请求方法，还有两个比较重要的概念：`安全与幂等。`

所谓的`"安全"是指请求方法不会"破坏"服务器上的资源，即不会造成实质性的修改。`
按照这个定义，只有GET和HEAD是安全的，因为它们是只读操作。

`"幂等"`实际上是一个数学用于，后来被借用到http协议里，意思是`多次执行相同的操作，
结果也是相同的。`即：多次"幂"后结果"相等"。   
很显然，GET 和 HEAD 既是安全的也是幂等的，DELETE可以多次删除同一个资源，效果都是“资源不存在”，
所以也是幂等的。     
POST和PUT会相对费解一点。POST 是“新增或提交数据”，多次提交数据会创建多个资源，
所以不是幂等的；而 PUT 是“替换或更新数据”，多次更新一一个资源，资源还是会第一次更新的状态，所以是幂等的。     
不过我们可以用sql来加深理解：POST类比insert，PUT类比update。就很清楚了。

## 小结

>`1、请求方法是客户端发出的、要求服务器执行的、对资源的一种操作；`    
>`2、请求方法是对服务器的“指示”，真正应如何处理由服务器来决定；`    
>`3、最常用的请求方法是GET和POST，分别是获取数据和发送数据；`    
>`4、HEAD方法是轻量级的GET，用来获取资源的元信息；`     
>`5、PUT基本上是POS的同义词，多用于更新数据；`     
>`6、“安全”与“幂等”是描述请求方法的两个重要属性，具有理论指导意义，可以帮助我们设计系统。`
                             









