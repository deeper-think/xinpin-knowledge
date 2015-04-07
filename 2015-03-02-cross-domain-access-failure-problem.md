---
layout: post
title: flashsocket 跨域访问失败问题
category: 案例分享
tags: [徐超]
keywords: Flash,Cache,缓存,跨域访问,跨域策略
description: 
---

> 我思故我在 -- 笛卡尔

## 问题现象
最近公司运维反馈有几个线上项目有用户投诉，说用户播放搜狐视频时被公司的缓存系统引导到缓存服务器后视频就无法播放了。查看缓存服务器的log并没有发现请求失败的日志，更奇怪的是log并没有来自用户的HTTP Get请求日志。到测试环境去测试发现很容易复现问题，客户端wireshark抓包分析客户端请求的数据流。

#### 用户请求引导阶段
播放搜狐视频过程中 flash player不断向源站发起视频资源的HTTP Get请求，HTTP Get请求被公司的缓存系统捕获到并发送302重定向数据包来引导用户请求到缓存系统，如下所示：

![用户发起视频资源的HTTP Get请求并被302重定向引导](http://7u2rbh.com1.z0.glb.clouddn.com/302redirect.jpg)

从上图可以确认用户请求引导阶段是正常的。

#### 用户请缓存系统发起请求阶段
接着分析用户向缓存服务器发起的请求及缓存服务器的响应，发现用户收到302重定向后没有直接向缓存服务器的80端口再次发起视频资源的HTTP Get请求，而是尝试向缓存服务器的843端口建立TCP连接，但是缓存服务器没有开启843端口，因此多次syn请求都被服务器rst掉了，如下所示：

![用户向缓存服务器843端口发起syn后被缓存服务器rst掉](http://7u2rbh.com1.z0.glb.clouddn.com/syn-rst.png)

再分析接下来的数据流，发现客户端flash player 再也没有发起视频资源的请求，至此大致发现问题所在。

## 相关原理
谷歌搜索[flash ，843]关键字，发现是flash 10以后引入的一个新的特性：

> flash 10以后引入了“跨域策略文件”，主要用于视频服务器对本机资源的访问权限进行控制。 flash跨域请求访问视频资源时首先会向视频服务器发起“跨域策略文件”请求，如果请求失败，则表示该视频服务器禁止任何第三方域的flash跨域请求。

## 解决方法
缓存服务器或者前端负载均衡服务器必须要监听843端口，当有客户端发起policy-file-request请求时，响应正确的cross-domain-policy文件给客户端flash player。用C语言写了一个简单的TCP服务器来实现该功能。-- [github 项目地址](https://github.com/deeper-think/flash-policy-serv)

enjoy the life !!!
