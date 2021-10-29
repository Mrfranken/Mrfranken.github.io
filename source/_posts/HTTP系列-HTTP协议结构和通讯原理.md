---
title: HTTP系列 - HTTP协议结构和通讯原理
date: 2018-04-13
tags:
- HTTP
category:
- 计算机网络
abstract: HTTP协议结构和通讯原理
---

## HTTP协议

### HTTP协议的特点
- **简单快速**：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快；
- **灵活**：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记；
- **无连接**：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间；
- **无状态**：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快
- 支持B/S及C/S模式。

### URL与URI的区别与联系
- URI： uniform resource identifier，统一资源标识符
- URL：uniform resource locator，统一资源定位器
- URN：uniform resource name，统一资源命名

[![GhndhR.jpg](https://s1.ax1x.com/2020/04/08/GhndhR.jpg)](https://imgchr.com/i/GhndhR)

我们在浏览器里输入的web地址是URL还是URI？

- URI可以分为URL，URN或同时具备locators和names特性的一个东西
- URN是唯一标识的一部分，是身份信息
- URL表示位置信息（定位资源）
- URL是URI的一种，但不是所有的URI都是URL
- URI和URL最大的差别是“**访问机制**”

### HTTP报文结构分析

#### 请求报文
[![Ghn091.jpg](https://s1.ax1x.com/2020/04/08/Ghn091.jpg)](https://imgchr.com/i/Ghn091)

HTTP的报文头大体可以分为四类
- 通用报文头            
[![Ghd1II.jpg](https://s1.ax1x.com/2020/04/08/Ghd1II.jpg)](https://imgchr.com/i/Ghd1II)
- 请求报文头  
[![Ghlisx.jpg](https://s1.ax1x.com/2020/04/08/Ghlisx.jpg)](https://imgchr.com/i/Ghlisx)
- 响应报文头             
[![GhlPQ1.jpg](https://s1.ax1x.com/2020/04/08/GhlPQ1.jpg)](https://imgchr.com/i/GhlPQ1)
- 实体报文头           
[![GhQzi4.jpg](https://s1.ax1x.com/2020/04/08/GhQzi4.jpg)](https://imgchr.com/i/GhQzi4)

#### 响应报文
[![GhlVoD.md.jpg](https://s1.ax1x.com/2020/04/08/GhlVoD.md.jpg)](https://imgchr.com/i/GhlVoD)


#### HTTP请求方法剖析
- HTTP/1.1常用方法
    - GET
        - GET方法用来请求访问已被URI识别的资源；
        - 指定的资源经服务器端解析后返回响应内容；
        - 也可以用来提交表单和其他数，比如 http://localhost/login.php?username=aa&pwd=123
    - POST
        - POST方法的主要目的不是获取响应主体的内容，一般用来传输实体的主体（提交表单）
    - PUT
        - 从客户端向服务器传送的数据取代指定的文档内容；
        - 与POST方法最大的不同是：PUT是幂等的，POST不是幂等的（理解幂等的概念
        参看https://blog.csdn.net/github_36032947/article/details/78421513）；
        - PUT方法更多时候用作传输资源；
    - HEAD(不作了解)
    - DELETE(不作了解)
    - OPTIONS
        - 用来查询针对请求URI指定的资源支持的方法
    - TRACE(不作了解)
        - 回显服务器收到的请求，主要用于测试或者诊断
    - CONNECT（不作了解）
        - 开启一个客户端与所请求资源之间的双向沟通的通道，它可以用来创建隧道
        
        
```
GET和POST的区别？
1. GET通过URL提交数据，数据在URL中可看到。POST提交数据时，数据放在HTTP请求的Body中；
3. GET请求有大小限制，POST请求没有；
3. 从安全性来看，POST对数据的保护性更好
```

### 响应状态码
用以表示网页服务器超文本传输协议响应状态的3位数字代码


状态码 | 英文名称 | 描述
-|-|-
200 | OK | 请求成功，请求所希望的响应或数据体将随此响应返回
202 | Accepted | 已接受，已经接受请求，但未处理完成
206 | Partial Content | 部分内容，服务器成功处理了部分GET请求 （断点续传）
301 | Moved Permanently | 永久移动，请求的资源已被永久的移动到新的URI，返回信息会包括新的URI。今后任何新的请求都应使用新的URI代替（永久重定向）
302 | Found | 临时移动，与301类似。但资源只是临时被移动。客户端应继续使用原有URI（临时重定向）
400 | Bad Request | 客户端请求的语法错误，服务器无法理解
401 | Unauthorized | 请求要求用户的身份认证
403 | Forbidden | 服务器理解请求客户端的请求，但是拒绝执行此请求
404 | Not Found | 服务器无法根据客户端的请求找到资源（网页）
500 | Internal Server Error | 服务器内部错误，无法完成请求
502 | Bad Gateway | 充当网关或代理的服务器，从远端服务器收到了一个无效的请求

### Cookie与Session
#### 什么是Cookie？
- Cookie实际上是小段的文本信息。客户端请求服务器，如果服务器需要记录该用户的状态，就向客户端浏览器颁发一个Cookie；
- 客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器。服务器检查该Cookie，以此来辨认用户状态。
> 打开一个网页，在浏览器的地址栏输入 javascript:alert(document.cookie) 可查看当前网页的Cookie

[![GbDI8U.md.jpg](https://s1.ax1x.com/2020/04/11/GbDI8U.md.jpg)](https://imgchr.com/i/GbDI8U)

#### 什么是Session？
- Session是另一种记录客户状态的机制，保存在服务器上。客户端浏览器访问服务器的时候，服务端把客户端信息以某种形式记录在服务器上；
- 客户端浏览器再次访问时只需要从Session中查找该客户的状态就可以了。
[![GbDo2F.md.jpg](https://s1.ax1x.com/2020/04/11/GbDo2F.md.jpg)](https://imgchr.com/i/GbDo2F)
- 保存Session ID的方式：
    - Cookie
    - URL重写
    - 隐藏表单
    
#### Cookie与Session的异同点
- 存放位置不同。Cookie保存在客户端，Session保存在服务端；
- 安全性（隐私策略）不同；
- 有效期不同（Cookie可以保存很长时间，服务器会清理过期的Session ID）；
- 对服务器造成的压力不同
 

### 常用请求头字段解析
[![Ghd8it.md.jpg](https://s1.ax1x.com/2020/04/08/Ghd8it.md.jpg)](https://imgchr.com/i/Ghd8it)
[![GhlpW9.md.jpg](https://s1.ax1x.com/2020/04/08/GhlpW9.md.jpg)](https://imgchr.com/i/GhlpW9)
[![Ghl9zR.md.jpg](https://s1.ax1x.com/2020/04/08/Ghl9zR.md.jpg)](https://imgchr.com/i/Ghl9zR)
[![GhlFL6.md.jpg](https://s1.ax1x.com/2020/04/08/GhlFL6.md.jpg)](https://imgchr.com/i/GhlFL6)
[![GhlAeK.md.jpg](https://s1.ax1x.com/2020/04/08/GhlAeK.md.jpg)](https://imgchr.com/i/GhlAeK)
[![GhlEdO.md.jpg](https://s1.ax1x.com/2020/04/08/GhlEdO.md.jpg)](https://imgchr.com/i/GhlEdO)
