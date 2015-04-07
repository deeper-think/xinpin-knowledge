---
layout: post
title: 01.squid 请求处理I
category: squid技术
tags: [徐超]
keywords: squid
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

#### 1. header\_access

配置描述:
保留或者删除回源的请求头和返回给客户端的响应头，默认保留所有的HTTP头。
配置格式:

    header_access header_name allow|deny [!]aclname

默认配置：

    none

#### 2. header\_replace

配置描述：
该配置项必须与header\_access配置项配合使用。要对某一个头选项做替换，则header\_access配置项要把该头选项配置为拒绝，默认是允许所有的头选项。

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
1、  对于header\_access中有定义的头选项优先级采用配置顺序规则，就是先匹>配最先配置的，如果不符合替换的规则，则再接着往下判断。
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

#### 3. req\_header\_access

配置描述：
其功能与header\_access相似，分频道规则也一样。
不同点在于：header\_access对回源的请求头和返回给客户端的响应头均起作用，
req\_header\_access只对回源的请求头起作用。
配置格式：

    req_header_access header_name allow|deny [!]aclname

可分频道配置。
默认配置：

    none

#### 4. rep\_header\_access

配置描述：
其功能与header\_access相似，分频道规则也一样。
不同点在于：rep\_header\_access只对返回给客户端的响应头起作用。
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

#### 5. clientreq\_header\_access

配置描述：
其功能与header\_access相似，分频道规则也一样。
不同点在于：header\_access对回源的请求头和返回给客户端的响应头均起作用，
clientreq\_header\_access只对客户端的请求头起作用。
跟req\_header\_access的区别在于，clientreq\_header\_access是在squid接到客户
端请求时就进行处理，而req\_header\_access则是在squid回上一级父或者源的时>候才处理。
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

#### 6. header\_name\_replace

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
做完头选项名称的替换之后，再执行header\_access的判断。在上例中，如果header\_access拒绝了X-Forward-For，由于已被替换成CLIENT-IP，则不会执行头选项
拒绝。如果header\_access拒绝了CLIENT-IP，则会执行头选项拒绝。
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
direction包含：CLI\_REQ, SER\_REQ, SER\_REP, CLI\_REP
如果method, direction出现其它字段(大小写不匹配)，则解析错误.
对于DEL操作，不需要message
对于ALT操作，message包括：
1）-r ,后跟两个部分，以空格分格，第一部分是正则规则，第二部分是新的内容.
2）直接跟新的内容.
注：
1）在单个方向上，新配置的处理顺序在旧有的配置（header\_access，req\_header\_access）的后面
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

### 功能描述:

除了HTTP头增删改之外的HTTP头处理，比如HTTP头检查等。 

### 相关配置:
    8. relaxed_header_parser
    9. check_repeated_header 

### 配置说明:
    
#### 8. relaxed\_header_parser

配置描述:
对HTTP头的解析，是否做严格检查，配置为on,warn,off. 
配置为on的时候，则做宽松检查，比如重复的头，HTTP头格式不对等都做忽略处理。
配置为warn,则再cache.log输出警告日志. 
配置为off,则认为是错误的HTTP头，HTTP头的内容会被重置。
配置格式:
    
    relaxed_header_parser on|warn|off 

不可分频道配置 
默认配置：

    on

备注:
A.      配置为warn，可能会导致输出大量的警告日志 
B.      配置为off，可能会因为HTTP头的异常，无法正常处理请求

#### 9. check\_repeated\_header

配置描述:
acl匹配头选项，是否检测重复的头选项。 
只对req\_header和rep_header两个acl类型生效。 
匹配的时候，是把两个头的值合并在一起判断。 
配置格式:

    check_repeated_header on|off
   
可分频道配置
默认配置：
   
    Off

配置例子:

    HOST www.test1.com 
	check_repeated_header off 
	acl test1_antileech_ref req_header Referer http://www\.test1\.com($|/.*)
	http_access deny !test1_antileech_ref 
	http_access allow all 
	HOST END
	pollurl -p 3128 http://www.test1.com/ --header="Referer:http://www.test1.com/\r\nReferer:http://www.test2.com/"允许访问
	pollurl -p 3128 http://www.test1.com/ --header="Referer:http://www.test2.com/\r\nReferer:http://www.test1.com/"拒绝访问 
		
	HOST www.test2.com 
	check_repeated_header off 
	acl test2_antileech_ref req_header Refererhttp://www\.test2\.com($|/.*) 
	http_access allow test2_antileech_ref 
	http_access deny all
	HOST END 

	pollurl -p 3128 http://www.test2.com/ --header="Referer:http://www.test2.com/\r\nReferer:http://www.test1.com/"允许访问
	pollurl -p 3128 http://www.test2.com/ --header="Referer:http://www.test1.com/\r\nReferer:http://www.test2.com/"拒绝访问 
	
	HOST www.test3.com
	check_repeated_header on 
	acl test3_antileech_ref req_header Refererhttp://www\.test3\.com($|/.*) 
	http_access deny !test3_antileech_ref
	http_access allow all 
	HOST END 
	pollurl -p 3128 http://www.test3.com/ --header="Referer:http://www.test3.com/\r\nReferer:http://www.test4.com/" 

该请求的Rerfer为http://www.test3.com/http://www.test4.com/，匹配acl规则，允许访问
 
	pollurl -p 3128 http://www.test3.com/ --header="Referer:http://www.test4.com/\r\nReferer:http://www.test3.com/" 

该请求的Referer为http://www.test4.com/http://www.test3.com/，匹配acl规则，允许访问

	HOST www.test4.com
	check_repeated_header on
	acl test4_antileech_ref req_header Refererhttp://www\.test4\.com($|/.*)
	http_access allow test4_antileech_ref
	http_access deny all 
	HOST END

	pollurl -p 3128 http://www.test4.com/ --header="Referer:http://www.test4.com/\r\nReferer:http://www.test3.com/"

该请求的Referer为http://www.test4.com/http://www.test3.com/,匹配acl规则，允许访问 

	pollurl -p 3128 http://www.test4.com/ --header="Referer:http://www.test3.com/\r\nReferer:http://www.test4.com/" 
该请求的Referer为http://www.test3.com/http://www.test4.com/,匹配acl规则，允许访问。

## 1.3. 请求/响应大小控制

### 功能描述:

控制请求/响应的大小

### 相关配置:

    10.   reply_header_max_size
    11.   reply_body_max_size 
    12.   request_header_max_size 
    13.   request_body_max_size

### 配置说明:

#### 10. reply\_header\_max\_size

配置描述:
用于配置允许的最大响应头大小,超过该大小则返回啥状态码，则返回601状态码，cache.log日志会打印”Too large reply header”. 
配置格式:

    reply_header_max_size20 KB

不可分频道配置 
默认配置：
20 KB

#### 11. reply\_body\_max\_size

配置描述:
用于配置允许的最大响应内容大小,如果响应头有Content-Length，则指示的大小超过该值，则拒绝访问，并提示”the request or replyis too large”，如果没有Content-Length的响应头，则当响应超过该大小的时候，则关闭连接。 
配置为0，表示不限制大小。 
配置格式:

    reply_body_max_sizebytes allow|deny acl acl... 

不可分频道配置
默认配置：

    0 allow all

#### 12. request\_header\_max\_size

配置描述:
用于配置允许的最大请求头大小,超过该大小则返回错误请求的响应，状态码为400 
配置格式:

    request_header_max_size20 KB

不可分频道配置 
默认配置：
    
    20 KB 

#### 13. request\_body\_max\_size

配置描述:
配置允许的请求body大小，如果请求body超过该限制，则返回请求错误的响应。当配置为0的时候，则表示不限制请求body大小。
配置格式:

    request_body_max_size bytes

不可分频道配置 
默认配置：
    
    0 KB 

## 1.4. URL处理

功能描述:
请求解析的url处理部分。 
相关配置

	14.   uri_whitespace 
	15.   allow_underscore 
	16.   check_hostnames  
	17.   acl_use_original_url 

配置说明：

#### 14. uri_whitespace

配置描述:
配置对url中包含空格的处理方式，配置strip遵循RFC2396协议，配置encode，遵循RFC1738协议。 
配置格式:

	uri_whitespacestrip|deny|allow|encode|chop 

不可分频道配置 
默认配置：

	strip 

#### 15. allow\_underscore

配置描述:
是否允许域名带下划线 
配置格式:

	allow_underscore on|off 

不可分频道配置 
默认配置:

	on
 
#### 16. check\_hostnames

配置描述:
是否做域名的合法性检查，为安全起见，并遵守协议，默认要做域名合法性检查。 
配置格式:

	check_hostnames on|off 
	
不可分频道配置 
默认配置:

	on 

#### 17. acl\_use\_original\_url

配置描述:
原来处理请求过程中，请求的url如果escape解码不通过，url会被截断，现在改为在解码不通过的情况下，不会对url进行截断。并新增配置，用于配置是否对url进行escape解码。 
此配置项为on的情况下，acl匹配之前不对url进行escape解码；为off情况，acl匹配之前先对url进行escape解码，解码不通过，不会将其截断。开关acl\_use\_original\_url为off就兼容旧的版本，在acl匹配之前先对url进行escape解码；此开关项打开的情况下，acl匹配之前不对url进行escape解码。以下acl类型需要使用这个开关配置项： 

	ACL_URL_REGEX 
	ACL_HASH_URL_REGEX 
	ACL_URLPATH_REGEX 
	ACL_URLLOGIN 

配置格式:

	acl_use_original_url on|off on|off 

不可分频道配置 
默认配置:

	off 

## 1.5. 去IP域名

### 功能描述:

需求背景：cache前端有haproxy，会重定向请求到cache，这时候，可能会在原来的请求基础上，多个cache的机器IP作为请求域名。Cache处理的时候，只有忽略该请求的IP域名方能正确处理请求。  
请求的url的域名和referer中的域名，如果域名是IP，就去掉这个域名。 
remove_iphost on 当且仅当Host:头是一个IP的时候有效，有效是指去除Host和referer中的IP。 
当url是http://ip/...的时候，并且referer也是http://ip/...的时候，referer的ip才会去掉，其他情况下只去掉url的ip，referer的ip不去掉，referer是http://ip的时候也不会去掉，后面再加个/就会了。 

### 相关配置:
    
    18.   remove_iphost 

### 配置说明

#### 18. remove\_iphost

配置描述:
用于配置是否去掉请求中的域名为IP的字段。 
配置格式:

	remove_iphost on|off 

不可分频道配置 
默认配置：

	off 

配置例子：
配置: 
	
    remove_iphost on 

则请求 
http://192.168.18.10/www.test.com/index.html变成http://www.test.com/index.html 
请求http://192.168.21.10/index.html则不会被去掉IP域名。 

	pollurl -p 3128  http://192.168.21.10/index.html --header="Referer:http://192.168.21.10/www.test.com/index.html" 

则Referer不会被去掉IP域名 

	pollurl -p 3128  http://192.168.21.10/www.test.com/index.html --header="Referer:http://192.168.21.10/www.test.com/index.html" 

请求url和Referer的IP域名都会被去掉。 

## 1.6. 客户端IP

### 功能说明：
1. 在没有做CDN之前，请求直接回源，源的服务器直接获取连接的IP地址当做客户端的IP，客户使用CDN之后，连接到源服务器的IP全部变为CDN节点的IP，直接获取连接地址的方式不正确，需要与CDN厂商协商一种机制，来获取客户端的IP。
2. CDN节点本身，前端也可能会有haproxy等7层交换软件，这样连接到CDN节点的地址也可能不是客户端的IP，CDN节点与前端代理软件同样需要一种机制来保证获取正确的客户端IP。

### 相关配置:
    
    19.   request_user_ip_header 
    20.   ws_x_forwarded_for 
    21.   forwarded_for 
    22.   acl_cdnsrc_with_src 
    23.   acl类型cdnsrc 
    24.   follow_x_forwarded_for 
    25.   acl_uses_indirect_client 
    26.   log_uses_indirect_client 

### 配置说明：

#### 19. request\_user\_ip\_header

配置描述:
用于配置获取客户端IP的请求头。 
配置格式:

	request_user_ip_header string 

不可分频道配置 
默认配置：

	Cdn-Src-Ip 

备注: 
1. 这里获取客户端IP的处理，只影响回源的IP选择，即影响select\_peer\_by\_user\_ip配置。 
2. select\_peer\_by\_user\_ip配置和回源选择的优先级关系，参考回源选择的相关文档 
配置例子：

	select_peer_by_user_ip allow all 
	request_user_ip_header X-Forwarded-For 

#### 20. ws\_x\_forwarded\_for
配置描述：
用于配置是否添加X-Forwarded-For头当做Cdn-Src-Ip使用，如果开启为on则按以下处理方式处理: 
1、对于一般连接： 
如果请求头中不包含X-Forwarded-For，则在回源的请求中加入X-Forwarded-For，以本地IP地址作为其值。 
如果请求中已经包含X-Forwarded-For，则在回源的请求中保留X-Forwarded-For的第一个值。 

2、对于长连接： 
如果请求中不包含X-Forwarded-For，则在回源的请求中加入X-Forwarded-For，值与此长连接第一个带X-Forwarded-For的请求的值一致。 
如果请求中已经包含X-Forwarded-For，则在回源的请求中保留X-Forwarded-For的第一个值。 
若配置为off，则会 
配置格式：

	ws_x_forwarded_foron|off 

可分频道配置。 
默认配置：

	off  

备注： 
此配置项会影响forwarded\_for配置处理结果。如果配置了ws\_x\_forwarded\_for，则forwarded\_for配置无效。 
配置例子：
配置: 

	ws_x_forwarded_for on 

测试过程: 

	pollurl http://www.test.com/ 

则，回源的请求头带: 
	
	X-Forwarded-For:127.0.0.1 

	pollurl http://www.test.com/ --header=” X-Forwarded-For:192.168.21.10,192.168.21.45”， 

则回源的请求头带: 

	X-Forwarded-For:192.168.21.10 

第一次请求， 

	telnet 127.0.0.1 80 
	GET / HTTP/1.0  
	Host: www.test.com 
	X-Forwarded-For:192.168.21.10,192.168.21.45 
	Proxy-Connection:keep-alive 

再发请求,  

	GET / HTTP/1.0 
	Host: www.test.com 
	X-Forwarded-For:192.168.21.119 
	Proxy-Connection:keep-alive 

则回源的请求中X-Forwarded-For:192.168.21.11 
  
再发请求 

	GET / HTTP/1.0 
	Host: www.test.com 
	Proxy-Connection:keep-alive 

则回源的请求中X-Forwarded-For:192.168.21.10 
以上功能如果ws\_x\_forwarded\_for off，则会按照http协议处理X-Forwarded-For。 

#### 21. forwarded\_for

配置描述:
用于配置是否追加本机的IP地址到X-Forward-For请求头中。 
forwarded_for默认为on，则会在X-Forwarded-For原有串之后添加本地IP，没有头选项的，则会添加头选项，并使用本地Ip作为值。 
如果配置forwarded\_for为off，则X-Forwarded-For保持原来的串回源，没有头选项的，也不添加新的头选项。 
配置格式：

	forwarded_for on|off 

不可分频道配置 
默认配置：

	on 

备注: 
ws\_x\_forwarded\_for会影响此配置处理结果。如果配置了ws\_x\_forwarded\_for，则forwarded\_for配置无效。 

#### 22. acl\_cdnsrc\_with\_src

配置描述：
在acl匹配的时候，是否将src的值作为Cdn-Src-Ip的值匹配acl配置。 
开关打开时，同时匹配srcip和CDN-SRC-IP头，开关关闭时，仅匹配CDN-SRC-IP头。 
影响acl配置类型cdnsrc的匹配。 
配置格式：

	acl_cdnsrc_with_src on|off 

可分频道配置。 
默认配置：

	off 

#### 23. acl类型cdnsrc
配置描述：
用于配置匹配Cdn-Src-Ip头的IP地址，在开启acl\_cdnsrc\_with\_src配置的情况下，如果没有Cdn-Src-Ip头，则用连接IP的值做acl匹配 
配置格式：

	acl aclname cdnsrc ip-address/域名 

可分频道配置。 
默认配置：
	
	无 

配置例子：
例1：

	acl testcdn cdnsrc 192.168.18.210 
	http_access deny testcdn 


	GET / HTTP/1.0 
	Host:bbs.xmu.edu.cn 
	Cdn-Src-Ip:192.168.18.45 
	允许 
  
	GET / HTTP/1.0 
	Host:bbs.xmu.edu.cn 
	Cdn-Src-Ip:192.168.18.210 
	拒绝 
 
例2：

	acl testcdn cdnsrc 192.168.18.210 121.14.88.14 
	GET / HTTP/1.0 
	Host:bbs.xmu.edu.cn 
	Cdn-Src-Ip:192.168.18.45 
	允许 
	GET / HTTP/1.0 
	Host:bbs.xmu.edu.cn 
	Cdn-Src-Ip:192.168.18.210 
	拒绝 
	GET / HTTP/1.0 
	Host:bbs.xmu.edu.cn 
	Cdn-Src-Ip: 121.14.88.14 
	拒绝 
  
例3：

	acl testcdn cdnsrc www.baidu.com 
	GET / HTTP/1.0 
	Host:bbs.xmu.edu.cn 
	Cdn-Src-Ip: 121.14.88.14 
	拒绝 
	GET / HTTP/1.0 
	Host:bbs.xmu.edu.cn 
	Cdn-Src-Ip:192.168.18.45 
	允许 

匹配源客户的IP选项配置:cdnsrc 
  
例4：

	acl url1 cdnsrc 192.168.21.10 
	http_access allow url1 

则请求头带有Cdn-Src-Ip: 192.168.21.10的请求才允许访问。 
  
例5：

	acl url1 src 192.168.21.10 
	http_access allow url1 

则从192.168.21.10发出的请求才允许访问。 
  
例6：

	acl url1 cdnsrc 192.168.21.10 
	http_access allow url1 
	acl_cdnsrc_with_src on 

则如果请求头包含Cdn-Src-Ip，则必须值为192.168.21.10的请求才允许访问。 
如果请求头不包含Cdn-Src-Ip，则从192.168.21.10发出的请求才允许访问。 
  
例7：

	acl allow_ip cdnsrc 192.168.21.10 
	HOST www.test1.com 
	acl_cdnsrc_with_src on 
	http_access allow allow_ip 
	http_access deny all 
	HOST END 
	HOST www.test2.com 
	acl_cdnsrc_with_src off 
	http_access allow allow_ip 
	http_access deny all 
	HOST END 

通过192.168.21.10机器作为客户端，访问这两个域名，则www.test1.com域名允许访问，www.test2.com域名拒绝访问。 

#### 24. follow\_x\_forwarded\_for
配置描述:
该配置功能未编译开启，没有启用，在编译开启FOLLOW\_X\_FORWARDED\_FOR时才有效。 

#### 25. acl\_uses\_indirect\_client
配置描述:
该配置功能未编译开启，没有启用，在编译开启FOLLOW\_X\_FORWARDED\_FOR时才有效。 

#### 26. log\_uses\_indirect\_client

配置描述:
该配置功能未编译开启，没有启用，在编译开启FOLLOW\_X\_FORWARDED\_FOR时才有效。

## 1.7. 循环检测

### 功能描述：
Squid原处理方式是出现一次via回环，则拒绝访问，考虑到咱们的特殊应用，可能因为配置出错导致回环，我们改变成以下的处理方式: 
1、在配置解析时，对cache\_peer中配置的节点域名或者IP进行检查，防止直接的循环. 

2、当出现循环时(间接，如A->B->C->A)，直接回源，并进行报警 

直接的回环： 
1）cache\_peer配置的是ip 
解析配置的时候，如果发现cache\_peer的ip和http\_port和本机的ip和端口一致，则重配置失败，返回1. 
2）cache\_peer配置域名 
peer的端口与本机的一致， 
dns服务器解析出来多个ip，则去除掉其中的本机ip， 
如果仅解析出来一个本机ip，则将peer设置为不可用的状态。 
下次，如果dns服务器解析出其他的ip，此peer仍然可用。 
间接循环的情况： 
第一次回环，直接回源，并进行报警。 
第二次回环（即机器名在Via头出现了两次），则返回403。 
### 相关配置:

	27.   via 
	28.   via_port2num_onoff 
	29.   replace_via_header 

配置说明：

#### 27. via
配置描述:
用于配置是否添加via头在请求/响应中，配置为on则表示添加，配置为off则不添加. 
配置格式:

	via on|off 

不可分频道配置 
默认配置：

	on 

#### 28. via\_port2num\_onoff

配置描述：
squid在回源头或返回头中，会在via头加入"机器名:端口号"，如"Via: xm223:8101" 
增加配置项(可以按频道配置),开启该配置项，则via中不加入端口号而是用数字编号(为了区分多进程中的不同squid)，如"Via: xm223:1" "Via: xm223:2" 
当配置该项的时候，在回源头和返回头中的via头域将把原先的“机器名：端口号”改为“机器名：数字”，其中这个数字对应map.conf中对应端口号的子进程id。 
配置格式：

	via_port2num_onoff on|off 

可分频道配置。 
默认配置：

	off 

配置例子：
（1）不打开via\_port2num_onoff开关和关闭via\_port2num\_on off时，响应头和回源头的via都是记录端口号。 
例子： 

	pollurl -p 8080 http://127.0.0.1/1.html， miss处理，响应头和回源的请求头都包含： 
	Via: 1.0 cache19.24:8080 (Cdn Cache Server V2.0)，记录是的端口号。 
再次发送该请求，hit处理，返回的响应头也包含上述via头部数据。 
（2）via\_port2num\_onoff  on 打开配置开关 
向squid请求：
	
	pollurl -p 8081 http://127.0.0.1/mytest.html， miss处理， 

回源的请求头部包含：
	
	Via: 1.0 cache19.24:1 (Cdn Cache Server V2.0)，不再是端口号，而是数字。 

向squid请求：
	
	pollurl -p 8082 http://127.0.0.1/index.html， miss处理,  

回源的请求头部包含：
	
	Via: 1.0 cache19.24:2 (Cdn Cache Server V2.0)，不再是端口号，而是数字。 

再次发送上面请求，Hit命中处理，via响应头同miss处理。 

#### 29. replace\_via\_header

配置描述：
需求背景:有的源对Via头的请求拒绝访问或者不返回压缩文件等情况: 
将Via头的名称替换成新的名称，没有配置则使用via头。 
使用此配置项，把Via的名称替换成其它名称。 
配置格式：

	replace_via_header string 

可分频道配置。 
默认配置：

	none，即根据via头判断 

备注： 
1、  要求配置的名称不能是可识别的头选项名称，如：Host, Last-Modified,Cache-Control等，否则，重配置失败。 
2、  via on/off开关仍然对新的头选项有效。 
3、  如果全局配置了replace\_via\_header X-Via，而频道内仍然希望使用Via，则在频道内配置replace_via_header none。 
配置例子：

	HOST www.test1.com 
	replace_via_header X-Via 
	HOST END 

## 1.8. HTTP/1.1

### 功能描述:
Squid是基于HTTP/1.0协议的，很多HTTP/1.1的功能不支持，故在没有任何配置情况下，squid采用HTTP/1.0回源，取得的响应直接返回给客户端。 
我们可配置，回源是否采用HTTP/1.1协议，及支持HTTP/1.1协议特有的处理，比如没有配置HTTP/1.1的情况下，会使用chunked编码的功能无法生效等等。 
### 相关配置:

	30.   use_http_1_1 
	31.   forward_request_version 

### 配置说明：

#### 30. use\_http\_1\_1

配置描述：
如果配置为on，则支持HTTP/1.1，客户端的请求使用HTTP/1.1协议，默认为持久连接，当出错的情况（比如返回大于等于400的状态码）时请求断开。对同一个url的HTTP/1.0请求和HTTP/1.1请求，当作不同的文件处理。分别使用HTTP/1.0和HTTP/1.1回源，并且缓存两个文件。 
如果squid没有配置use_http\_1\_1，或者配置use\_http\_1\_1 off，则不论客户端使用HTTP/1.0或者HTTP/1.1协议，默认都为非持久连接的。 
配置格式：

	use_http_1_1 off|on 

可分频道配置。 
默认配置：

	off: 即使用HTTP/1.0回源，返回版本为HTTP/1.0。 

备注： 
1.squid从低版本升级到此版本，squid中缓存的都是HTTP/1.0的返回文件，而客户的请求大部分是HTTP/1.1（目录的浏览器大部分使用HTTP/1.1），会导致缓存中的文件无法使用而大量回源，升级的时候要谨慎。 
2.版本回退时，新版本缓存的HTTP/1.1文件无法再使用，也会产生同样的问题。 
3.对大部分文件，请求都是HTTP/1.1，因此只缓存一个HTTP/1.1的文件。 
4.PURGE的时候，不管使用的是HTTP/1.0还是HTTP/1.1，会将缓存中的该URL的文件全部删除。 
5.即使配置use_http\_1\_1 off，PURGE也会将两个版本的缓存都清除。 
配置例子：

	HOST www.test.com 
	use_http_1_1 on 
	HOST END 

#### 31. forward\_request\_version
配置描述：
更改squid回源请求的HTTP版本（默认是HTTP/1.0），不论请求时1.0版本还是1.1版本，都使用HTTP/1.1回源. 
配置为1表示HTTP/1.1, 0表示HTTP/1.0 
备注: 非0也表示使用HTTP/1.1 
配置格式：

	forward_request_version 0|1 

可分频道配置。 
默认配置：

	0 

配置例子：

	HOST www.test.com 
	forward_request_version 1 
	HOST END 

## 1.9. 访问控制

### 访问控制: 
通过相关的access的配置，用于配置请求或者响应是否允许访问。 
绕过防盗链: 
构造一些特殊的请求，这些请求能通过所有的防盗链验证(配置中的http\_access，http\_access2，redirect\_url，query\_access\_url，query\_access\_ip)。这些特殊的请求中，加入了一个MD5码，这个MD5由“uri + 加密串 + 客户IP”计算得到。MD5码放在请求的一个自定义的头域中，uri不包含？及后面的内容。 
客户IP是小写的16进制格式，网络字节顺序，固定长度8，如127.0.0.1就是0100007f请求头的名称和加密串可以配置。 
如果配置了以下的all\_access\_header、all\_access\_key这两个配置项。一个请求有指定头，并且头中的MD5正确，则这个请求不受配置项http\_access，http\_access2，redirect\_url，query\_access\_url，query\_access\_ip的影响。 

### 相关配置:

	32.   http_access 
	33.   http_access2 
	34.   http_reply_access 
	35.   miss_access 
	36.   all_access_header 
	37.   all_access_key 
	38.   icp_access 
	39.   htcp_access 
	40.   htcp_clr_access 
	41.   ident_lookup_access 

### 配置说明：

#### 32. http\_access
配置描述：
通过访问列表来限制用户是被允许或被拒绝访问。 
配置格式：

	http_access allow|deny [!]aclname ... 

可分频道配置。 
默认配置：

	deny all 

备注： 
如果没有匹配到访问列表行，则默认值取访问列表行最后一行的相反值。比如：如果最后一行是deny，则默认值是allow 
所以为避免潜在错误，一般要根据实际使用情况在访问列表行的最后写上http\_access allow all或http\_access deny all 
注意： 

	http_access allow acl1 acl2 

不等价于 

	http_access allow acl12  
	http_access allow acl2 

前者需同时满足两个acl规则，后者只需满足一个即可。 

#### 33. http\_access2

配置描述：
与http\_access配置项类似，通过访问列表来限制用户是被允许或被拒绝访问。 
区别是: 该配置在请求改写后做访问控制检查。 
如果没有设置此配置项，那么只有http\_access起作用。 
配置格式：

	http_access2 allow|deny [!]aclname ... 

可分频道配置。 
默认配置：

	无 

#### 34. http\_reply\_access
配置描述：
用于配置该响应是否可用于响应客户请求。http\_reply\_access与http\_access类似。不同之处在于，http\_access在请求进来的时候，就判断是否允许访问;而该配置在取得响应的时候才判断是否允许访问。 
大部分访问控制基于客户请求的方式，对这些使用http\_access就够了。而某些时候，需要基于响应的内容来判断是否允许访问的情况，比如基于响应内容类型来允许或拒绝请求。 
配置格式：

	http_reply_access allow|deny [!] aclname ... 

可分频道配置。 
默认配置：

	allow all 

#### 35. miss\_access
配置描述：
miss_access用于配置squid在缓存MISS的时候是否回源。 
配置格式：

	miss_accessallow|deny [!]aclname ... 

不可分频道配置。 
默认配置：

	无 

配置例子

	acl localclients src 172.16.0.0/16 
	miss_access allow localclients 
	miss_access deny all 

以上配置则表示，内网172.16.的请求在缓存MISS的时候可以回源。而其他客户端只能取到已缓存的响应，对于没有缓存的响应，则返回403响应。 

#### 36. all\_access\_header

配置描述：
配置绕过防盗链配置，需要检查的HTTP请求头域的名称，与all\_access\_key配置一起使用。 
配置格式：

	all_access_header string 

不可分频道配置。 
默认配置：

	WSEC 

#### 37. all\_access\_key

配置描述：
配置加密串，与all\_access\_header配置一起使用。如果配置了该配置，则会检查请求头中是否包含all\_access\_header配置字符串，如果包含，再判断值与加密串算出来的是否一致，如果一致，则表示允许访问，绕过所有其他的访问控制检查。 
加密串的计算方法，详见功能描述。 
配置格式：

	all_access_key string 

不可分频道配置。 
默认配置：

	none 

配置例子：

	all_access_header WSKEY 
	all_access_key ws-secret-key 
	http_access allow all 
	HOST www.test1.com 
	http_access deny all 
	HOST END 

测试过程： 

	pollurl -p 3128  http://www.test1.com/index.html，拒绝访问，返回403响应。 
	# echo -n /index.htmlws-secret-key0100007f|md5sum 
	6558d49616b7d73272aa2ccd21d9bc1a  - 
	# pollurl -p 3128  http://www.test1.com/index.html  --header="WSKEY:6558d49616b7d73272aa2ccd21d9bc1a" 
	允许访问。 

#### 38. icp\_access
配置描述：
假如你的squid被配置来服务ICP响应，那么应该使用icp\_access列表。squid默认拒绝所有ICP查询，必须使用icp\_access规则列表，允许来自邻居cache的查询。 
配置格式:

	icp_access allow|deny [!]aclname ... 

不可分频道配置。 
默认配置：

	deny all 

备注: 
该配置没有在使用。 

#### 39. htcp\_access
配置描述：
基于访问列表来决定访问HTCP端口时是否被允许或被拒绝 
配置格式:

	htcp_access  allow|deny [!]aclname ... 

不可分频道配置。 
默认配置：
   
	deny all 

备注: 
该配置没有在使用。 

#### 40. htcp\_clr\_access

配置描述:
基于访问列表来决定清除内容使用HTCP时是否被允许或被拒绝 
配置格式：

	htcp_clr_access allow|deny [!]aclname ... 

不可分频道配置。 
默认配置：

	deny all 

配置例子：

	#Allow HTCP CLR requests from trusted peers 
	acl htcp_clr_peer src 172.16.1.2 
	htcp_clr_access allow htcp_clr_peer 

备注: 
该配置没有在使用。 

#### 41. ident\_lookup\_access

配置描述:
它允许你对某些请求执行ident查询。squid默认不进行ident查询。假如请求被ident\_lookup\_access规则（或ident ACL）允许，那么squid才会进行ident查询。 
配置格式:

	ident_lookup_access allow ident_aware_hosts 
	ident_lookup_access deny all 

不可分频道配置。 
默认配置：

	deny all 

配置例子：

	acl ident_aware_hosts src 198.168.1.0/255.255.255.0 
	ident_lookup_access allow ident_aware_hosts 
	ident_lookup_access deny all 

备注: 
该配置没有在使用。 

## 1.10. 限制用户使用IP访问

### 相关配置:

	42.   access_deny_ip 
	43.   acl的allowip类型 

### 配置说明:

#### 42. access\_deny\_ip

配置描述：
是一个开头选项，为on时则打开IP访问限制功能。 
配置格式：

	access_deny_ip on|off 

不可分频道配置。 
默认配置：

	off 

#### 43. acl的allowip类型

配置描述：
是新添加的acl类型，可以设置允许使用访问的IP地址，比如IP为127.0.0.1和本机IP 
可与access\_deny\_ip配合使用。 
配置格式：

	acl aclname allowip ip-address 

不可分频道配置。 
默认配置：

	无 

配置例子：

	access_deny_ip on 
	acl mylocalIP allowip 127.0.0.1 
	http_access deny !mylocalIP 

表示允许用IP访问http：//127.0.0.1的网页，不允许其它的IP地址访问 。
