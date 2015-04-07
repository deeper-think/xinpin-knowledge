---
layout: post
title: haproxy基于url对HTTP请求进行封锁
category: 案例分享
tags: [徐超]
keywords: haproxy
description: 
---

> 我思故我在 -- 笛卡尔

## 问题现象
公司运维反馈线上缓存服务器缓存了一些域名下的涉黄图片资源，想在负载均衡这一层对用户的请求进行过滤，将某些涉黄url的请求直接响应403。线上负载均衡使用的是haproxy软件， haproxy支持基于url对HTTP请求进行封堵, 如下：

    acl test hdr(host) www.baidu.com                     #基于访问域名 
    acl test url https://www.baidu.com/?tn=61089049_pg   #基于请求url
    acl test url_reg ^http://.*baidu.*$                  #基于请求url正则
    block if test                                        #对满足test的请求进行封堵

然而线上配置对URL:

    http://7u2rbh.com1.z0.glb.clouddn.com/openssl%20complie%20failed.png

进行封堵，测试机上绑定7u2rbh.com1.z0.glb.clouddn.com域名的解析到指定的LB，测试机测试发现并不能封堵，然而配置浏览器代理的方式进行测试，可以正常封堵：

    acl test url http://7u2rbh.com1.z0.glb.clouddn.com/openssl%20complie%20failed.png
    block if test

## 问题分析
在测试机上抓包比较域名解析绑定和指定http代理这两种方式下测试机发出的http请求的差别，发现问题所在：

![不用HTTP GET请求](http://7u2rbh.com1.z0.glb.clouddn.com/get-req.png)

## 解决方法
针对两种类型的GET请求都进行封堵，正常配置如下：

    acl test url http://7u2rbh.com1.z0.glb.clouddn.com/openssl%20complie%20failed.png
    acl test url /openssl%20complie%20failed.png
    block if test


enjoy the life !!!
