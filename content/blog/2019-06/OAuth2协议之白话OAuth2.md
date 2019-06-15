---
date: "2019-06-15T16:52:48+08:00"
draft: false
title: "OAuth2协议之白话OAuth2"
tags: ["oauth2"]
categories: ["杂技浅尝"]
toc: true
---

## OAuth2协议是什么？
OAuth2通常也叫OAuth2.0。在我们日常生活中，我们常常会用微信或者QQ等账号去注册或者登陆其他应用。比如：`用微信玩王者荣耀`等。     
这类在应用程序上，通过第三方账号授权，提供有限的用户数据的行为。就是OAuth2协议的一种最常见的使用场景。

### 官方术语
>`OAuth2`是开放授权的一个标准，旨在让用户允许第三方应用去访问改用户在某服务器中的特定私有资源，
而可以不提供其在某服务器的账号密码给到第三方应用。

值得注意的是：OAuth2.0是OAuth协议的延续版本，但不向后兼容OAuth 1.0，即完全废止了OAuth1.0。

## OAuth2原理
### 简单推导
这里我不直接介绍OAuth2的原理。我们用一个简单的推导过程，来引出OAuth2协议的核心思想。    
我们假定一个场景：    

>有个app，一个该app的用户想用微信登陆此app。那么对于该app来说，可能他需要去获取用户的微信昵称和头像等信息。
即此`app（客户应用）需要访问用户的微信（资源服务器）数据`。

{{% center %}}<img name="Finder-toolbox" src="/images/blog/2019-06/oauth2_01.png"  width='400px'/>{{% /center %}}

那如果我们是微信，要提供用户数据，可能会怎么做？我们可以提供一个`API`，当有用户请求的时候，将数据返回客户应用。
{{% center %}}<img name="Finder-toolbox" src="/images/blog/2019-06/oauth2_02.png"  width='400px'/>{{% /center %}}

但是此时会有一个问题，如果来了一个恶意的客户服务器怎么办？
{{% center %}}<img name="Finder-toolbox" src="/images/blog/2019-06/oauth2_03.png"  width='400px'/>{{% /center %}}

这个时候就需要在服务器加上安全保护机制。业界实践是提出给客户应用颁发一个`Access Token`。它表示客户应用被授权可以访问用户数据。
客户服务器访问的时候带上这个令牌，服务器接收到请求之后，拿到令牌进行校验，校验通过再返回数据。
{{% center %}}<img name="Finder-toolbox" src="/images/blog/2019-06/oauth2_04.png"  width='400px'/>{{% /center %}}

但是这时候又引出来问题：Access Token是怎么颁发的呢？谁来颁发呢？    
这时我们就需要引入一个概念：`授权服务器`。它用来管理和颁发Access Token。
{{% center %}}<img name="Finder-toolbox" src="/images/blog/2019-06/oauth2_05.png"  width='400px'/>{{% /center %}}

但是，什么情况授权服务器会颁发Access Token呢？在真实情况下，在`颁发token前需要征询用户同意`。
{{% center %}}<img name="Finder-toolbox" src="/images/blog/2019-06/oauth2_06.png"  width='400px'/>{{% /center %}}

上面所推导出来的就是oauth2最简单也是最常用的一个流程了。

### 正式定义
#### OAuth协议历史
大致开始于2007    
2010 – RFC5849定义了OAuth1.0。    
2010 – IETF开始OAuth2.0制定工作。    
2012年中 – 第一作者和编辑退出， 并将其名字从所有规范中删除(戏剧性)。    
2012年10月 - RFC6749（授权框架）、RFC6750（token规范）出炉。

#### 四个角色
`客户应用(Client Application)`：通常是一个Web或者无线应用，它需要访问用户的受保护资源。    
`资源服务器(Resource Server)`：是一个web站点或者web service API，用户的受保护数据保存于此。    
`授权服务器(Authorized Server)`：在客户应用成功认证并获得授权之后，向客户应用颁发访问令牌Access Token。    
`资源拥有者(Resource Owner)`：资源的拥有人， 想要分享某些资 源给第三方应用。    

#### 三个术语
`客户凭证(Client Credentials)`：客户的clientId和密码用于认证客户。    
`令牌(Tokens)`：授权服务器在接收到客户请求后，颁发的访问令牌。    
`作用域(Scopes)`：客户请求访问令牌 时，由资源拥有者 额外指定的细分权 限(permission)。 

#### 总结
再回到文章开篇的问题：OAuth2.0协议是什么？我们可以总结为这几点：

>• 用于`REST/APIs`的代理授权框架(delegated authorization framework)。    
>• 基于令牌`Token`的授权，在无需暴露用户密码的情况下，使应用能获取对用户数据的有限访问权限。    
>• 解耦认证和授权。    
>• 事实上的标准安全框架，支持多种用例场景：        
 -服务器端WebApp    
 -浏览器单页SPA    
 -无线/原生App    
 -服务器对服务器之间

注：`令牌Token是OAuth2.0的核心`。它是用户给应用授予有限的访问权限，让应用能够代表用户去访问用户的数据。    
也就是说：`OAuth2.0的本质就是如何获取Token，如何使用Token。`

## OAuth2容易误解的地方       
OAuth仅支持HTTP协议        
OAuth并没有定义授权处理机制。    
OAuth并没有定义token格式。    
OAuth2.0并没有定义加密方法。    
OAuth2.0并不是单个协议，而是一套协议。    
`OAuth2.0是一个授权协议，而不是一个认证协议。`认证的逻辑都是使用者自己实现的。