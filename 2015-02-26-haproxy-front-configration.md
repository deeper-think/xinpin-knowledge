---
layout: post
title: haproxy 前端配置
category: haproxy技术
tags: [徐超]
keywords: haproxy
description:
---

> 我思故我在 -- 笛卡尔

## 1.1. HTTP头处理(增删改)

### 功能描述:

可配置对HTTP头进行增删改，支持客户请求头，回源请求头，回源响应头，客户>响应头4个方向的操作。

### 相关配置:

    1. header_access:
    2. header_replace
    3. req_header_access
    4. rep_header_access1
    5. clientreq_header_access
    6. header_name_replace
    7. header_control

### 配置说明：

#### 1. header_access

配置描述:
保留或者删除回源的请求头和返回给客户端的响应头，默认保留所有的HTTP头。
配置格式:

    header_access header_name allow|deny [!]aclname

默认配置：

    none

#### 2. header_replace

配置描述：
该配置项必须与header_access配置项配合使用。要对某一个头选项做替换，则header_access配置项要把该头选项配置为拒绝，默认是允许所有的头选项。

    header_replace header_name message allow aclname

表示符合aclname规则的请求都允许作此替换，其他的均不做该替换。

    header_replace header_name message deny aclname:

表示符合aclname规则的请求都不作此替换，其他的则作替换。
配置格式：

    header_replace header_name message allow|deny aclname

可分频道配置。
默认配置：
无
备注：
1、  对于header_access中有定义的头选项优先级采用配置顺序规则，就是先匹>配最先配置的，如果不符合替换的规则，则再接着往下判断。
配置例子：
例1：

    header_access Content-Type deny all
    header_replace Content-Type text/plain

这里使用原配置，表示拒绝Content-Type头选项，然后对该头选项使用text/plain做替换，对所有的请求都作此替换。
例2：

    acl test1 dstdomainwww.test1.com
    acl test2 dstdomain www.test2.com
    header_access Content-Type deny all
    header_replace Content-Type text/plain allow test1
    header_replace Content-Type imag/plain allow test2

表示拒绝Content-Type头选项，然后如果该URL的域名是www.test1.com，则对该>头选项使用text/plain做替换。对于域名为www.test1.com的则使用imag/plain做
替换。对于其他域名的不做替换（返回的内容将不包含Content-Type头选项）。
例3：

    acl test1 dstdomain www.test1.com
    header_access Content-Type deny all
    header_replace Content-Type text/plain deny test1

对于www.test1.com的Content-Type不作此替换，返回的头将不包含Content-Type选项。其他的请求返回的响应将包含配置内容的Content-Type头。

#### 3. req_header_access

配置描述：
其功能与header_access相似，分频道规则也一样。
不同点在于：header_access对回源的请求头和返回给客户端的响应头均起作用，
req_header_access只对回源的请求头起作用。
配置格式：

    req_header_access header_name allow|deny [!]aclname

可分频道配置。
默认配置：

    none

#### 4. rep_header_access

配置描述：
其功能与header_access相似，分频道规则也一样。
不同点在于：rep_header_access只对返回给客户端的响应头起作用。
配置格式：

    rep_header_access header_name allow|deny [!]aclname

可分频道配置。
默认配置：

    none

der_access New-Header deny all

则回源的请求头和响应头中均不会有New-Header。

    req_header_access New-Header deny all

则回源的请求头中不包含New-Header，但响应头中会有New-Header

    rep_header_access New-Header deny all

则响应的头中不包含New-Header，但回源的请求头中会有New-Header。

#### 5. clientreq_header_access

配置描述：
其功能与header_access相似，分频道规则也一样。
不同点在于：header_access对回源的请求头和返回给客户端的响应头均起作用，
clientreq_header_access只对客户端的请求头起作用。
跟req_header_access的区别在于，clientreq_header_access是在squid接到客户
端请求时就进行处理，而req_header_access则是在squid回上一级父或者源的时>候才处理。
配置格式：

    clientreq_header_access header_name allow|deny [!]aclname

可分频道配置。

默认配置：

    none

配置例子：
例1：

    header_access New-Header deny all

则回源的请求头和响应头中均不会有New-Header。

    clientreq_header_access New-Header deny all

则会去掉客户端的请求中的New-Header头选项，再进行其它处理。

例2：

    clientreq_header_access Accept-Encoding deny all

则认为客户端的请求中不带Accept-Encoding头选项，于是，返回给客户端非压缩
的文件。
这时候如果需要回源，回源的请求也不会带Accept-Encoding

#### 6. header_name_replace

配置描述：
用于将回源请求中的头选项名称进行替换，但头选项内容不变。而且不会对返回>给用户端的头选项名称替换。
配置格式：

    header_name_replace srcname dstname allow/deny aclname
    header_name_replace srcname dstname

可分频道配置。
默认配置：

    none

备注：
1、  如果有多个配置，则按顺序匹配，若前面的acl已经匹配，则后面的不起作>用。
2、  当全局配置与分频道配置同时存在时，以分频道中的配置才有效。
配置例子：
例1：

    header_name_replace X-Forward-For CLIENT-IP allow all

则会将回源请求中的：X-Forward-For: 127.0.0.1改成：CLIENT-IP: 127.0.0.1

例2：

    header_name_replace X-Forwarded-For CLIENT-IP deny acl1
    header_name_replace X-Forwarded-For MY-IP allow acl2
    header_name_replace X-Forwarded-For BB-IP4

则：对于acl1，其应该被替换成BB-IP, 其它则被替换成CLIENT-IP。对acl2的配>置无效。
做完头选项名称的替换之后，再执行header_access的判断。在上例中，如果header_access拒绝了X-Forward-For，由于已被替换成CLIENT-IP，则不会执行头选项
拒绝。如果header_access拒绝了CLIENT-IP，则会执行头选项拒绝。
如果原来dstname就存在的话，则会删除dstname头选项，然后再执行替换。
若配置对dstname再次执行替换，将会无效。
例如：

    header_name_replace X-Forwarded-For BB-IP
    header_name_replace BB-IP  X-Forwarded-For

如果先把X-Forwarded-For头替换成BB-IP头，则不会再次把BB-IP头替换成 X-Forwarded-For头。
如果先把BB-IP替换成X-Forwarded-For，则不会再次把X-Forwarded-For头替换成
BB-IP头。
至于，先执行哪个替换，取决于X-Forwarded-For和BB-IP在请求头中的位置。

例3：

    HOST www.test.com3
    header_name_replace X-Forwarded-For BB-IP
    HOST END
    header_name_replace Cdn-Src-IP Content-Type

则：对于 www.test.com频道，会把X-Forwarded-For替换成BB-IP，但不会对Cdn-Src-IP进行替换其它频道，则会把Cdn-Src-IP替换成Content-Type。

#### 7. header_control

配置描述:
该配置可以对HTTP头的4个方向进行增删改，可分频道配置。
配置格式:

    header_control method direction headername [message] allow/deny aclname

其中，
method包含：ADD， ALT, DEL
direction包含：CLI_REQ，SER_REQ, SER_REP, CLI_REP
如果method, direction出现其它字段(大小写不匹配)，则解析错误.
对于DEL操作，不需要message
对于ALT操作，message包括：
1）-r ,后跟两个部分，以空格分格，第一部分是正则规则，第二部分是新的内容.
2）直接跟新的内容.
注：
1）在单个方向上，新配置的处理顺序在旧有的配置（header_access，req_header_access）的后面
2）此配置在4个方向上的处理不相互影响
3）配置仅影响头的处理过程，并没有修改缓存文件中的头内容
配置格式:

    header_control method direction headername [message] allow/deny aclname.

可分频道配置
默认配置:

    none

配置例子:

    acl denyacl req_header User-Agent aaa
    http_access deny denyacl
    HOST aaa.aa.com
    header_control ADD CLI_REQ User-Agent aaa allow all
    HOST END
    HOST bbb.bb.com
    header_control ALT CLI_REQ User-Agent aaa allow all
    HOST END
    HOST ccc.cc.com
    header_control DEL CLI_REQ User-Agent allow all
    HOST END
    HOST ddd.dd.com
    header_control ALT CLI_REQ User-Agent -r ^Fire.* aaa allow all
    HOST END
    HOST eee.ee.com
    header_control ALT CLI_REQ User-Agent -r ^op(.*) Op$ allow all
    HOST END

则：
请求aaa.aa.com/, User-Agent: bbbb, 允许访问
请求：aaa.aa.com/，拒绝访问
请求bbb.bb.com/，允许访问
请求bbb.bb.com/, User-Agent: bbbb, 拒绝访问
请求ccc.cc.com/ 允许访问
请求ccc.cc.com/，User-Agent: aaaa，允许访问
请求ddd.dd.com/, User-Agent: bbbb，允许访问
请求ddd.dd.com/，User-Agent: FireFox，拒绝访问
请求eee.ee.com/, User-Agent: opera，允许访问
请求eee.ee.com/，User-Agent: aaa，拒绝访问

## 1.2. HTTP头处理(其他)


