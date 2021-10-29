---
title: HTTP系列 - HTTP协议的一些特性（二）
date: 2018-04-19
tags:
- HTTP
category:
- 计算机网络
abstract: HTTP协议的一些特性（二）
---

### HTTP缓存

#### 为什么要使用HTTP缓存？
- 减轻服务器压力
- 加快客户端访问速度



#### 缓存的内容是什么？
一些静态资源文件，例如：css、js文件


#### HTTP缓存头部字段

<table>    
  <tr><th>请求/响应投</th><th>作用</th>
  <tr><td rowspan="20">Cache-Control</td></tr>
  <tr><td>no-store 所有内容都不缓存</td></tr>
  <tr><td>no-cache 缓存，但是浏览器使用缓存前，都会请求服务器判断缓存资源是否是最新</td></tr>
  <tr><td>max-age=x(单位秒) 请求缓存后的X秒不再发起请求，优先级高于Expires</td></tr>
  <tr><td>public 客户端和代理服务器(CDN)都可缓存</td></tr>
  <tr><td>Expires <b>响应头</b>，代表资源过期时间，有服务器返回提供。是HTTP1.0的属性，在与max-age共存的情况下，优先级要低</td></tr>
  <tr><td>Last-Modifies <b>响应头<b/>，资源最新修改时间，由服务器告诉浏览器</td></tr>
  <tr><td>if-Modified-Since <b>请求头</b>,资源最新修改时间，由浏览器告诉服务器，实际上就是上一次响应服务器给的Last-Modified。和Last-Modified是一对，它两会进行对比</td></tr>
  <tr><td>Etag <b>响应头</b>，资源标识，由服务器告诉浏览器</td></tr>
  <tr><td>if-None-Match <b>请求头</b>，缓存资源标识，由浏览器告诉服务器，实际上就是上一次响应服务器给的Etag。和Etag是一对，它两会进行对比</td></tr>
</table>   


[![GOd8V1.md.jpg](https://s1.ax1x.com/2020/04/12/GOd8V1.md.jpg)](https://imgchr.com/i/GOd8V1)
描述：可以看到在这个场景中，Expires起到了标识资源过期时间的作用。

问题:当Expires已经过期，当浏览器再次请求服务器，而f.js文件并没有改变，name这次请求其实仍然可以使用缓存中的f.js。那怎么避免Expires过期的情况下，浏览器再次请求f.js呢？继续看场景二。


[![GOdcPf.md.jpg](https://s1.ax1x.com/2020/04/12/GOdcPf.md.jpg)](https://imgchr.com/i/GOdcPf)
描述：服务器通过在响应头中加入 Last-Modified 字段，告诉浏览器等Expires规定的时间过期了你再次访问时），我们就对比 Last-Modified（文件修改时间）字段就可以了。（浏览器再次发送请求时发送的其实是if-Modified-Since字段，其值与Last-Modified一致）。看个例子

[![GOwPJK.md.jpg](https://s1.ax1x.com/2020/04/12/GOwPJK.md.jpg)](https://imgchr.com/i/GOwPJK)

问题：那问题又来了，Last-Modifies 字段只能精确到秒，假设文件是在一秒内发生变动的，那在这种情况下浏览器拿到的就不是最新的文件（例如电商平台中时时刻刻有商品变化的情况）。那怎么办呢？继续看场景三。

[![GOBrZ9.md.jpg](https://s1.ax1x.com/2020/04/12/GOBrZ9.md.jpg)](https://imgchr.com/i/GOBrZ9)

描述：现在的情况就是在max-age和Etag有的情况下，Expires和Last-Modified字段的优先级是比较低的。当Max-age=60时，浏览器在60秒内不会向服务器发起请求。当60过后，浏览器带上if-Modified-Since 和 if-None-Match 发起请求。服务器会比对if-Modified-Since 和 if-None-Match，尽管此时也传送了 if-Modified-Since， 但在有 if-None-Match 的情况下会优先比较 if-None-Match 字段，因为这个字段更精准。看个例子

[![GOrktI.md.jpg](https://s1.ax1x.com/2020/04/12/GOrktI.md.jpg)](https://imgchr.com/i/GOrktI)

其实上面的方案还是有些缺陷，当Expires或max-age不过期的情况下，浏览器是无法感知服务器文件变化的，那怎么解决呢？


### 内容协商

#### 内容协商机制

> 指客户端和服务端就响应的资源内容进行交涉，然后提供给客户端最为合适的资源。
> 内容协商会以响应资源的语言，字符集，编码方式等最为判断的基准。

#### 内容协商的方式
- 客户端驱动：客户端发起请求，服务器发送可选项列表，客户端作出选择后再发送第二次请求
- 服务器驱动：服务器检查客户端的请求头部集并决定提供哪个版本的页面

#### 服务器驱动内容协商-请求头部集

请求头 | 含义
---|---
Accept | 告知服务器发送何种媒体类型
Accept-Language | 告知服务器发送何种语言
Accept-Charset | 告知服务器发送何种字符集
Accept-Encoding | 告知服务器发送何种编码



响应头 | 含义
---|---
Content-Type | 对应 Accept
Content-Language | 对应 Accept-Language
Content-Encoding | 对应 Accept-Encoding