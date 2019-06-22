---
date: "2019-06-22T23:00:48+08:00"
draft: false
title: "https工作原理与配置"
tags: ["nginx", "https", "计算机网络"]
series: ["nginx"]
categories: ["nginx"]
toc: true
---

## https配置
在前面的一篇博客[《使用nginx搭建一个简单具有反向代理功能的网站》](https://xp329486175.github.io/blog/2019-06/%E4%BD%BF%E7%94%A8nginx%E6%90%AD%E5%BB%BA%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E5%85%B7%E6%9C%89%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E5%8A%9F%E8%83%BD%E7%9A%84%E7%BD%91%E7%AB%99/)中我们已经搭建了一个具有反向代理的网站。那如果我们要接着配置https应该怎么做呢？

如果是自己的web网站，我们可以使用这样一个自动化脚本进行配置：`Let’s Encrypt`。    

具体命令：
```shell
#yum install python2-certbot-nginx
#cerbot —nginx—nginx-server-root=/Users/xiepeng/xp/nginx/conf -d a.test.pub
```
这两行命令实际的作用就是：获取ssl证书，然后在nginx.conf中添加相应配置：
```shell
listen       443;

ssl on;
ssl_certificate /etc/letsencrypt/live/xxx/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/xxx/privkey.pem;
```
## TLS和SSL协议
自动化脚本配置是很简单的，但是配置不是目的，重要的学习其中工作原理。

HTTPS并非是应用层的一种新协议。只是HTTP通信接口部分使用SSL（Secure Socket Layer）/TLS（Transport Layer Security）协议替代而已。
所以，HTTPS是身披SSL/TLS外壳的HTTP。
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/ssl_01.png" width='600px'/>{{% /center %}}
ssl协议是1995年网景公司推出的，在3.0获得非常大的发展。     
微软把ie和windows捆绑推出后，导致网景遇到了很大的困景，应微软要求，改名为tls。     
ssl/tls协议主要工作在表示层，通过握手、交换密钥、警告、对称加密等方式，让http层在无感知的情况下完成加密。      

## 对称加密和非对称加密
我们直接用一张图来看对称加密和非对称加密的关系。
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/ssl_02.png" width='600px'/>{{% /center %}}
### 对称加密

>bob和alice持有同一把密钥。怎么加密就怎么解密。    

### 非对称加密
  
>持有一对密钥，公钥和私钥。一般有两种应用场景：    
`数据加密`：如果bob要给alice传递数据。bob通过alice的公钥加密，然后把密文传给alice。alice通过自己的私钥进行解密。     
`身份验证`：有一段信息alice通过私钥加密，把密文发送给bob，bob通过alice给的公钥解密，只要能解密，就说明该密文是alice发出的。

## PKI公钥基础设施
我们申请证书和使用证书的流程一般如下图：
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/ssl_03.png" width='500px'/>{{% /center %}}

如图：      

>1、证书订阅人向CA(公信机构)发起申请，CA颁发证书。      
2、证书订阅人拿到证书以后将证书部署到服务器。     
3、当浏览器访问https站点的时候，浏览器会去请求证书，nginx等web服务器会把公钥证书发给浏览器️，
浏览器会去验证书是不是合法和有效的。      
4、CA中心会把过期证书放到CRL服务器，会把过期的证书形成一条链条。这种方式性能非常差，优化成了ocsp的方式。      
5、OCSP可以单个证书查询，浏览器可以直接去查询ocsp相应程序。而且nginx有个ocsp开关，可以主动查询。

## 证书类型
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/ssl_04.png" width='400px'/>{{% /center %}}
证书类型一般分为DV、OV、EV三种。

>DV：只要是你申请的域名归属正确，也就是域名指向的是你正在申请证书的那台服务器就可以申请成功。实时、免费。    
OV：验证填写企业或者机构是正确的。要几天、收费。    
EV：更严格，浏览器显示更友好。    

这三种类型，其实效果是一样的，唯一区别就是证书链。     
证书链：一般分为三级，根证书（操作系统/浏览器级别）- 二级证书 - 主证书。这里就不展开了。    
一般nginx向浏览器发送证书只要发送主证书和二级证书。

通常我们去ca申请的证书也被称为自签名证书，SSL使用证书来创建安全连接。有两种验证模式：

>仅客户端验证服务器的证书，客户端自己不提供证书；    
客户端和服务器都互相验证对方的证书。    

显然第二种方式安全性更高，一般用网上银行会这么搞，但是，普通的Web网站只能采用第一种方式。

## TLS通讯过程
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/ssl_05.png" width='400px'/>{{% /center %}}
TLS通讯过程一般分为四步：

>验证身份    
 达成安全套件共识    
 传递密钥    
 加密通讯
 
 具体到图中每一步就是：    
 1、浏览器发送信息到服务器。（主要告诉服务器，浏览器或者客户端支持哪些加密算法）     
 2、服务器根据自己支持的加密算法做选择。    
 3、根证书验证。（同时把公钥证书和证书链发送给浏览器）    
 4、发送done。     
 5、客户端生成自己的私钥和发给服务器的公钥。      
 6、服务器根据自己的私钥和客户端发送的公钥生成密钥。（客户端和服务器生成的密钥是相同的，非对称加密算法来保证的）    
 7、服务器和客户端各自用自己生成的密钥进行数据加密，进行通信。     
 
 总结说来：TLS通讯过程就是在做两件事：`交换密钥、加密数据。`