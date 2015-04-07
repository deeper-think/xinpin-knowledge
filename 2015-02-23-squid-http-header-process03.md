---
layout: post
title: 01.squid 请求处理III
category: squid技术
tags: [徐超]
keywords: squid
description:
---

> 我思故我在 -- 笛卡尔

## 1.21. 诺基亚下载信息反馈

无用配置项。

## 1.22. 域名及端口替换

### 功能描述:

配置内部重定向功能之后,流量会被记录到目标的域名上去.

	acl flv1bnneteaseRedirectUrl url_regex -i ^http://flv1\.bn\.netease\.com\.lxdns\.com/.*. 
	interior_redirector http://flv1.bn.netease.com.lxdns.com/(.*) http://flv1.bn.netease.com/$1 allow flv1bnneteaseRedirectUrl 

需要另外配置流量采集:

	mgr_traffic flv1.bn.netease.com flv1.bn.netease.com.lxdns.com ^http://flv1\.bn\.netease\.com\.lxdns\.com/.* 

流量应该默认记录到原始域名上. 
  
术语定义： 
A域名：客户端请求的原始域名，即加速域名 
B域名：缓存文件的域名
C域名：回源解析的域名 
D域名：回源请求的HOST域名，不包括回父的域名 
E域名：日志记录的域名 
F域名：流量统计的域名

### 相关配置:

	141.  domain_replace_store 
	142.  domain_replace_dst_host 
	143.  domain_replace_log 
	144.  domain_replace_traffic 
	145.  domain_replace_traffic_regx
	146.  domain_replace_dst
	147.  domain_replace_dst_regx
	148.  replace_origin_port

### 配置描述:

#### 141. domain\_replace\_store

配置描述：
用于替换缓存文件的域名.
配置格式：

	domain_replace_store newdomain [allow/deny aclname] 

默认配置：

	无

#### 142. domain\_replace\_dst\_host

配置描述：
替换回源请求的HOST域名，不包括回父的域名, cache\_peer中增加一个配置选项fu\_server，用于标识此peer是我们自己架设的父，而不是客户的源.
该配置可配置域名和端口 
配置格式：

domain\_replace\_dst\_host newdomain [allow/deny aclname]

默认配置：

	无

配置例子:

	HOST aaa.aa.com
	domain_replace_dst a.testa.com
	HOST END
	HOST bbb.bb.com
	domain_replace_dst a.testa.com
	HOST END
	HOST ccc.cc.com
	domain_replace_dst a.testa.com:80
	HOST END 

1）请求http://aaa.aa.com/,则回源a.testa.com，81端口 
2）请求http://aaa.aa.com：8080/,则回源a.testa.com，81端口 
3）请求http://bbb.bb.com/,则回源a.testa.com，80端口 
4）请求http://bbb.bb.com：8080/,则回源a.testa.com，8080端口 
5）请求http://ccc.cc.com/,则回源a.testa.com，80端口 
6）请求http://ccc.cc.com：8080/,则回源a.testa.com，80端口

#### 143. domain\_replace\_log

配置描述：
替换日志记录的域名 
配置格式：

	domain_replace_log newdomain [allow/deny aclname] 

默认配置：

	无 

#### 144. domain\_replace\_traffic
配置描述：
替换流量统计的域名。 
配置格式：

	domain_replace_traffic newdomain [allow/deny aclname] 

默认配置：

	无 

#### 145. domain\_replace\_traffic\_regx

配置描述:
根据配置规则自动解析出流量统计域名
配置格式:

	domain_replace_traffic_regx [-i] old_domain_regx new_domain acl_rules

-i 为可选项，是否区分old\_domain\_regx大小写，使用 -i 则不区分大小写 
old\_domain\_regx为要匹配的正则表达式 
new\_domain正则表达式匹配后得出的替换域名.
acl-\_rules为acl匹配规则.
可分频道配置 
默认配置:

	none 

配置例子:

	traffic_auto_add_host on
	HOST www.qq.com 
	domain_replace_traffic_regx http://([^/]+)/([^/]+)/.*  $2.$1 allow all
	HOST END 

当访问http://www.qq.com/bm1/a.html的时候，统计流量的域名为bm1.www.qq.com 
当访问http://www.qq.com/bm2/a.html的时候，统计流量的域名为bm2.www.qq.com 
当访问http://www.qq.com/bm3/a.html的时候，统计流量的域名为bm3.www.qq.com
注意：

 1）只有traffic\_auto\_add\_host 配置为 on的时候，该配置项才会生效 
 2）如果同时配置domain\_replace\_traffic时，该配置项无效

#### 146. domain\_replace\_dst
配置描述：
当配置此配置项时，请求在回源时，进行域名解析，会查询newdomain的ip地址，但是回源的请求中的HOST还是原来的域名。 
配置格式：

	domain_replace_dst newdomain [allow/deny aclname]
	domain_replace_dst newdomain 

如果没有配置acl规则，则对整个频道起作用。 
可分频道配置。 
默认配置：

	无

#### 147. domain\_replace\_dst\_regx
配置描述:
此配置的功能与domain\_replace\_dst一样，区别在于，可以进行正则匹配。
如果某个频道内有配置domain\_replace\_dst，则此配置无效。
配置格式：

	domain_replace_dst_regx olddomain newdomain

可分频道配置。
默认配置：

	无

配置例子：
配置： 

	HOST aaa.aa.com 
	domain_replace_store bbb.bb.com 
	HOST END 

过程：
1）取aaa.aa.com的请求，则B，C，D，F域名均使用bbb.bb.com域名，E域名使用aaa.aa.com域名，此时功能与内部重定向功能一致。
2）增加配置： 
	HOST bbb.bb.com
	domain_replace_dst ccc.cc.com 
	domain_replace_dst_host ddd,dd,com 
	domain_replace_log eee.ee.com
	domain_replace_traffic fff.ff.com 
	HOST END 

清空缓存，取aaa.aa.com的请求，则B域名使用bbb.bb.com，C域名使用ccc.cc.com，D域名使用ddd,dd,com，E域名使用eee.ee.com，F域名使用fff.ff.com 
3）再增加配置 
	
	conf_from_orig_host on 

清空缓存，取aaa.aa.com的请求，则B域名使用bbb.bb.com，C，D，E，F域名均使用aaa.aa.com 

4）此时，如果要得到步骤2的效果，则应该把它的配置转移到HOST aaa.aa.com里面

#### 148. replace\_origin\_port

配置描述:
用于替换回源端口和动态源探测端口 
配置格式:

	replace_origin_port port 

可分频道配置 
默认配置:
        无

## 1.23. 回源带城市信息

### 功能描述:

回源的请求可配置添加客户端IP所属的城市信息。 

### 相关配置:

	149.  forward_header_add_client_city 

### 配置说明:

#### 149. forward\_header\_add\_client\_city

配置描述:
当使用forward\_header\_add\_client\_city配置了城市信息头时，回源的请求会添加该请求头。
配置格式:

	forward_header_add_client_city header-name 

可分频道配置
默认配置:

无

配置例子:

	HOST aaa.aa.com 
	forward_header_add_client_city X-Client-Ip-City
	HOST END 

## 1.24. POST处理

### 功能描述:

包括记录POST数据到指定文件、POST加速

### 相关配置:

	150.  post_log 
	151.  post_log_allow 
	152.  post_log_size 
	153.  post_tcp_recv_bufsize 
	154.  hash_post_len_allow 
	155.  broken_posts 

### 配置说明:

#### 150. post\_log

配置描述：
配置记录POST数据的文件路径。
配置格式：

	post_log path

不可分频道配置。 
默认配置：

	/usr/local/squid/var/logs/post.log 

#### 151. post\_log\_allow

配置描述：
配置是否启用记录post数据日志的功能。 
配置格式：

	post_log_allow on|off

可分频道配置。 
默认配置：

	off

#### 152. post\_log\_size

配置描述：
配置记录post数据的长度。当实际POST数据大于该大小的时候，只记录配置的长度，小于该大小的时候，全部记录。单位为“字节”。 
配置格式：

	post_log_size int 

可分频道配置。 
默认配置：

	1024 

配置例子：

	post_log /usr/local/squid/var/logs/post_data.log
	HOST www.aaa.com 
	post_log_allow on
	post_log_size 10
	HOST END

表示记录POST数据到/usr/local/squid/var/logs/post_data.log。对www.aaa.com启用记录POST数据功能，记录的数据长度为10字节，小于10字节时全部记录，大于10字节时只记录10字节。 

#### 153. post\_tcp\_recv\_bufsize

配置描述：
用于配置POST请求的buffer大小。 
配置值小于等于0的时候，则使用系统默认的buffer大小。
配置格式：

	post_tcp_recv_bufsize bytes 

可分频道配置。
默认配置：

	0 

配置例子：

	HOST www.suning.com 
	post_tcp_recv_bufsize 20 MB 
	HOST END 

则表示对域名www.suning.com，POST请求数据上传的buffer大小为20MB。 

#### 154. hash\_post\_len\_allow
配置描述：
当其打开时，如果POST请求中不包含Content-Length也允许访问。原先版本对于POST请求中不包含Content-Length的，拒绝请求，状态码为411。 
配置格式：

	hash_post_len_allow on|off 

可分频道配置。 
默认配置：

	off，即不允许POST请求中不包含Content-Length。% x: l( E* l1 t6 a

配置例子：
例1：
若配置
	
	hash_post_len_allow on 

发送
	
	POST /template/index/index_news_list.jst.html HTTP/1.1
	Host:   cn.aigomusic.com 

则允许访问 
例2：
若配置
	
	hash_post_len_allow off 

发送 
	
	POST /template/index/index_news_list.jst.html HTTP/1.1 
	Host:   cn.aigomusic.com

则拒绝访问 

发送

	POST /template/index/index_news_list.jst.html HTTP/1.1 
	Host:   cn.aigomusic.com 
	Content-Length: 0

允许访问.

#### 155.     broken\_posts
配置描述：
对于满足此配置的请求，squid会在PUT/POST请求的body后加一个额外的CRLF。某些不遵守协议的HTTP服务器依赖于此额外的CRLF。
配置格式：

	broken_posts allow/deny aclname 

不可分频道配置。 
默认配置：

	none

配置例子：

	acl buggy_server url_regex ^http:// 
	broken_posts allow buggy_server

## 1.25. 预取

###　功能描述:

一个典型的HTML页面（称为基页）会包含一些外部资源（比如图片、文本文件等，称为嵌入页），浏览器请求这个页面时，也会依次请求这些外部资源。如果squid在响应对基页的请求时主动回源获取基页中包含的嵌入页的内容并缓存，当浏览器请求这些嵌入页时，squid就可以直接从缓存中获取内容返回给浏览器，省去了回源的时间，提高了响应速度。 
说明： 
①嵌入URL不同于页面中的链接，链接需要用户点击才会请求，嵌入页会由浏览器自动请求，我们预取的是后者； 
②只预取html页面中img/frame/iframe/script/embed标签的src属性包含的URL； 
③不嵌套预取：预取的HTML页面中的嵌入URL不再预取；
④预取的URL支持重定向和页面改写功能（预取除外），不支持其他功能； 
⑤对于预取的页面，如果页面本身可以缓存，按照它自己的缓存设置缓存，如果页面不能缓存，则按照prefetch\_cache\_time的配置强制缓存； 
⑥当预取和链接替换一同启用时，先进行链接替换，后进行预取。
⑦只对HTTP/1.1请求生效。 

### 相关配置:

	156.  link_prefetch2
	157.  prefetch_concurrency
	158.  prefetch_cache_time 

### 配置说明:

#### 156. link\_prefetch

配置描述：
配置对哪些页面进行预取。 
配置格式：

	link_prefetch allow/deny aclname

可分频道配置。 
默认配置：

	none

#### 157. prefetch\_concurrency

配置描述：
配置预取时每个基页所能发送预取请求的最大并发数。如使用默认配置时，一个基页最多可以同时发送5个预取请求，两个基页最多可以同时发送10个预取请求。

配置格式：

	prefetch_concurrency int 

不可分频道配置。
默认配置：

	5 

#### 158. prefetch\_cache\_time

配置描述：
配置预取的文件缓存时间，只对原来不可缓存的文件生效。应配置在基页的频道下面，对此基页包含的预取URL有效。
配置格式：

	prefetch_concurrency 5 seconds

可分频道配置。 
默认配置：

	5 seconds

配置例子：
	
	prefetch_concurrency 3 
	HOST aaa.aa.com 
	link_prefetch allow all
	prefetch_cache_time 5 minute 
	HOST END 

对于aaa.aa.com域名下的所有html页面开启预取功能，若预取的URL不可缓存，则强制缓存5分钟。对同一个基页，每次最多可以同时发送3个预取请求。


## 1.26. 页面改写

### 功能描述:

此部分包含以下功能： 
（1）去空格和注释 
（2）链接替换 
（3）内容替换 
（4）嵌入URL替换# o9 N. C) V2 p7 W3 T
以下是各功能的详细描述： 
(1) 删除空格及注释：
1、不删除标签里的空格，只删除数据（即标签外的其它部分）里的空格； 
2、pre、script、style、textarea四个标签之间的数据保持原样，空格和注释均不删除；
3、删除空格的行为：多个空格（包括空格和\t）被合并为一个空格，多个换行符（包括\r，\n和\f）被合并为一个换行符（\n），换行符后的所有空格均删除；
4、以<!--[if开头的注释不删除，<!--<![endif]-->注释不删除；
5、删除空格和删除注释须同时进行，无法单独进行。
(2) 替换URL： 
1、将满足正则匹配的URL替换为指定的URL； 
2、只替换embed、img、iframe、frame、script标签的src属性包含的URL； 
3、当URL以http://开头时直接匹配，当U不以http://开头时在URL前加上当前页面所在的路径然后进行匹配； 
4、替换结构体为一个链表，即：可以指定多个正则表达式和相应的替换字符串，按照链表序进行替换，替换一次即停止。 
(3) 替换特殊字符串： 
1、将满足正则匹配的字符串替换为指定的字符串； 
2、只替换数据里的字符串； 
3、script标签之间的内容不进行替换. 
4、可进行多次替换。 
(4) 嵌入URL替换： 
在一个典型的HTML页面中，会包含一些外部资源（比如图片、文本文件等），用户请求这个页面时会同时请求这些资源，并将它们缓存到本地。当用户刷新此页面时，会再次发出对这些静态资源的http请求。如果这些文件没有过期，服务器会返回304。“嵌入URL改写”的目的就是避免客户端对这些没有过期的文件发出请求，节约发出请求到接收响应之间的时间，从而减少客户端页面的加载时间。
具体做法是，改写HTML页面中的嵌入URL，使其包含资源的特定信息（称为pv值），当服务器修改资源时，资源的URL也将改变（体现为pv值改变），缓存中的旧资源将不会被使用。同时重写HTTP响应头，使资源可以在用户本地缓存1年。比如：

	HTML tag: <img src="images/logo.gif"/>
	HTTP header: Cache-Control:public, max-age=300

将被修改为： 
	
	HTML tag: <img src="images/logo.gif;pv=12345678"/> 
	HTTP header: Cache-Control:public, max-age=31536000 

squid继续使用原来的max-age值（上例中为300秒）检查资源是否改变，如果改变，则资源的pv值也会随之改变，新的URL将会被提供给用户。当资源没有改变时，用户使用本地缓存，从而避免了对这些资源的请求。 

说明：
1、目前，页面优化实现的功能除字符串替换外仅针对于html页面，因此，在page\_rewrite\_content\_type中配置了其它类型时，除text/html之外的其他类型均不会生效。标记为text/html类型的页面，开头如无doctype声明亦不会生效（防止对某些声明为text/html类型但实际内容为script代码的响应进行修改）。对于字符串替换功能，参考page\_replace\_content\_fulltext配置项的说明。 
2、页面改写只对HTTP/1.1的请求有效，对HTTP/1.0的请求无效。因为页面改写总是返回chunk响应。
3、对于同样的功能，只有离源最近的节点上的配置生效。例如，源返回的响应依次经过A、B到达客户端，A上配置替换字符串，B上配置替换字符串和替换URL，只有A上配置的替换字符串功能生效，即使B配置的内容不同（如A上替换word\_1到word\_2，B上替换word\_3到word\_4），B上的替换字符串功能也不生效。由于B之前的节点（A）没有配置替换URL，所以B上的替换URL功能生效。
4、内部重定向A→B，页面改写的配置应放在B频道内，开启conf\_from\_orig\_host on时应放在A频道内。
5、嵌入URL改写对可以缓存的页面有效，对不能缓存的页面无法生效。

###　相关配置:

	159.  page_remove_space_remark 
	160.  page_replace_link_str 
	161.  page_replace_content_str 
	162.  page_rewrite_content_type 
	163.  page_replace_content_fulltext 
	164.  page_rewrite_embedded_url 
	165.  pv_key 
	166.  embedded_url_filetype 
	167.  embedded_url_rewrite_factor 

### 配置说明:

#### 159. page\_remove\_space\_remark

配置描述：
配置对哪些页面进行去空白和注释的操作。 
配置格式：

	page_remove_space_remarkallow/deny aclname 

可分频道配置。
默认配置：

	none

#### 160. page\_replace\_link\_str
配置描述：
配置URL替换的规则。 
配置格式：

	page_replace_link_str http://www.ggssl.com/(.*) http://www.google.com/$1 allow/deny aclname

可分频道配置。 
默认配置：

	none

#### 161. page\_replace\_content\_str
配置描述：
配置字符串替换的规则。
配置格式：

	page_replace_content_str QQ PP allow/deny aclname 

可分频道配置。 
默认配置：

	none

#### 162. page\_rewrite\_content\_type

配置描述：
配置页面改写的文档类型。
配置格式：

	page_rewrite_content_typeext/html text/css text/javascript text/plain 

可分频道配置。 
默认配置：

	text/html 

#### 163. page\_replace\_content\_fulltext

配置描述：
配置是否强制全文替换，只对字符串替换有效，开启时无论是否html页面都强制全文替换，未开启时html页面只替换显示内容，非html页面全文替换。 
配置格式：

	page_replace_content_fulltext on|off 

可分频道配置。 
默认配置：

	off 

配置例子：

	HOST aa.aaa.com. J' R+ H: Z7 T) S- y6 c
	page_remove_space_remark allow all 
	page_replace_content_str (.*)MP3(.*) $1MP4$2 allow all 
	page_replace_link_str (.*)img1.cache.netease.com(.*) $1img1.netease.com$2 allow all 
	page_rewrite_content_type text/html text/css text/javascript text/plain 
	page_replace_content_fulltext on 
	HOST END 

对aa.aaa.com域名下的html页面，进行去空格、去注释操作，对于页面中embed、img、iframe、frame、script标签的src属性包含的URL，如果满足正则表达式(.*)img1.cache.netease.com(.*)，则将其替换为$1img1.netease.com$2。由于配置了page_replace_content_fulltext on，对于类型属于text/html、text/css、text/javascript、text/plain的页面，把页面中所有满足(.*)MP3(.*)的字符串替换为$1MP4$2。 

#### 164. page\_rewrite\_embedded\_url

配置描述：
配置对哪些页面做304改写。 
配置格式：

	rewrite_embedded_url allow/deny aclname 

可分频道配置。 
默认配置：

	none

#### 165. pv\_key

配置描述：
配置改写嵌入页url时，增加的后缀。 
配置格式：

	pv_key string 

可分频道配置。
默认配置：

	;pv=

#### 166. embedded\_url\_filetype

配置描述：
解析的嵌入页的格式，满足格式的URL才改写。 
配置格式：

	embedded_url_filetype .jpg .jpeg .gif .png 

可分频道配置。 
默认配置：

	.jpg .jpeg .gif .png .mpg .mpeg .wmf .bmp .emf .avi .xbm .ico3 c, O# ?: I( O3 c# e4 g

#### 167. embedded\_url\_rewrite\_factor

配置描述：
多于等于embedded\_url\_rewrite\_factor个嵌入URL需要被改写的时候才改写页面。
配置格式：

	embedded_url_rewrite_factor N 

可分频道配置。 
默认配置：

	15

配置例子：

	HOST aa.aaa.com 
	acl pvstr url_regex -i .*;pv=.+ 
	interior_redirector ^(.*);pv=.+$ $1 allow pvstr 
	page_rewrite_embedded_url allow all 
	pv_key ;pv= 
	embedded_url_filetype .jpg .jpeg .gif .png 
	embedded_url_rewrite_factor 1 
	HOST END 

对于aa.aaa.com域名下的html页面开启嵌入URL预取功能。对嵌入URL的文件类型为jpg、jpeg、gif、png的进行改写，改写字符串为;pv=。当页面中需要改写的URL多于等于1个时才改写，否则不改写。 

## 1.27. 关键字替换

### 功能描述:

提供三种替换方式： 
（1）用同一个字符替换关键字中开头的多个字符。
（2）用一个字符串替换关键字，当字符串长度大于关键字的长度，则截取关键字长度的字符串做替换，当字符串长度小于关键字的长度，截取关键字长度的字符串做替换。 
（3）把一组关键字中的每一个随机替换为另一组字符串中的任意一个。
其中，前两个功能由配置replaced\_keywords、keyword\_replace\_char、keyword\_replace\_len实现，第3个功能由配置keyword\_list、replace\_string\_list实现。（1）（2）的功能和（3）不兼容，配置了（3）的功能时，会自动屏蔽（1）（2）的功能。 

### 相关配置:

	168.  keyword_replace_log 
	169.  keyword_replace_access 
	170.  replaced_keywords 
	171.  keyword_replace_uncompress_onoff 
	172.  keyword_replace_char 
	173.  keyword_replace_len 
	174.  keyword_list 
	175.  replace_string_list 

### 配置说明:

#### 168.     keyword\_replace\_log

配置描述：
配置替换日志的路径。
配置格式：

	keyword_replace_logpath 

不可分频道配置。 
默认配置：

	none

#### 169. keyword\_replace\_access

配置描述：
配置对哪些页面进行关键词替换。 
配置格式：

	keyword_replace_accessallow/deny aclname

可分频道配置。
默认配置：

	none 

#### 170. replaced\_keywords

配置描述：
被替换的关键词集合。其中的每一个都要被替换。 
配置格式：
	replaced_keywords 1.1.1.1 1.2.3.4 5.6.7.8

不可分频道配置。
默认配置：

	none

#### 171. keyword\_replace\_uncompress\_onoff

配置描述：
开启此配置时，返回未压缩的响应。 
配置格式：

	keyword_replace_uncompress_onoff on|off 

可分频道配置。 
默认配置：

	off 

#### 172. keyword\_replace\_char
配置描述：
配置用于替换的字符串。当其长度为1时keyword_replace_len配置生效，当其长度大于1时keyword_replace_len配置无效。
配置格式：

	keyword_replace_char string

不可分频道配置。 
默认配置：
	
	* 

#### 173. keyword\_replace\_len

配置描述：
配置应该替换的长度。当keyword\_replace\_char长度大于1时，此配置无效。当keyword\_replace\_char长度等于1时，对关键字从头开始使用keyword\_replace\_char配置的字符替换此配置配置的次数。
配置格式：

	keyword_replace_len int

不可分频道配置。

默认配置：
	
	100

配置例子：
例1：

	replaced_keywords 192.168.21.224 192.168.21.225 
	replaced_keywords 192.168.21.239
	keyword_replace_char A 
	keyword_replace_len 3

对于出现的192.168.21.224、192.168.21.225和192.168.21.239，使用字符“A”替换，替换3次，即替换为AAA.168.21.224、AAA.168.21.225和AAA.168.21.239。 
例2：

	replaced_keywords 192.168.21.224 192.168.21.225
	replaced_keywords 192.168.21.239
	keyword_replace_char 1.1.1.1 
	keyword_replace_len 3 //该配置无效 

替换长度小于关键字，则会把192.168.21.224替换成1.1.1.1.21.224,其他IP类似。 
例3：

	replaced_keywords 192.168.21.224 192.168.21.225 
	replaced_keywords 192.168.21.239 
	replaced_keywords 61.23.53.1
	keyword_replace_char 172.135.134.110 
	keyword_replace_len 3 //该配置无效 

替换长度大于关键字，则会把192.168.21.224替换成172.135.134.11,其他IP类似。
其中，61.23.53.1会被替换成172.135.13。

#### 174. keyword\_list

配置描述：
被替换的关键词集合。其中的每一个都要被替换。 
配置格式：

	keyword_list 1.1.1.1 1.2.3.4 5.6.7.8 

不可分频道配置。
默认配置：

	none 

备注：
关键字匹配算法为最短匹配。 

#### 175. replace\_string\_list
配置描述：
用来替换的关键词集合。每次替换时，选其中的任意一个来替换。和keyword_list配合使用。 
配置格式：

	keyword_list 1.1.1.1 1.2.3.4 5.6.7.8

不可分频道配置。
默认配置：

	none 

备注： 
以上两个配置的功能受到配置page\_replace\_content\_fulltext的影响。对于非html页面，进行全文替换，对于HTML页面，如果配置page\_replace\_content\_fulltext on，则进行全文替换，否则只替换显示的部分。

配置例子：
例1：

	keyword_list 192.168.21.254 192.168.21.25 192.168.21.2
	replace_string_list 1.2.3.4 5.6.7.8 9.10.11.12 13.14.15.16 17.18.19.20
	keyreplace_log /usr/local/squid/var/logs/keyword_replace.log 
	  
	HOST aa.aaa.com 
	acl html url_regex -i aa.aaa.com/.*\.html$ 
	keyword_replace_access allow html
	keyword_replace_access deny all 
	page_rewrite_content_type text/html
	HOST END

对于aa.aaa.com域名下以html结尾的URL且文档类型为text/html的，把其中出现的每一个192.168.21.254、192.168.21.25、192.168.21.2都替换为1.2.3.4、5.6.7.8、9.10.11.12、13.14.15.16、17.18.19.20中的任意一个。
例2：

	keyword_list 192.168.21.2 192.168.21.227 
	replace_string_list 1.2.3.4

按照以上配置， 192.168.21.227将被替换为1.2.3.4。 

## 1.28. ws iweb视频转码页面改写

### 功能描述:

根据不同的url模式在获取的HTML页面内</body>标签前添加一段字符串：
 
	<script type="text/javascript" src="iweb_js_path"></script>
	<script language="javascript">
	var url = "web_service_url";
	js_entry_func(url, "player_div_id", rtts_site_addr); 
	</script> 

字符串中加粗的部分根据配置项给出的配置进行替换。

### 相关配置：

	176.  iweb_page_rewrite
	177.  iweb_js_path 
	178.  rtts_site_addr
	179.  js_entry_func 
	180.  web_service_url 
	181.  player_div_id 

以下五个配置项必配：iweb\_js\_path，rtts\_site\_addr，js\_entry\_func，web\_service\_url，player\_div\_id，如果有一项或多项缺失，或者没有符合acl匹配规则的项，则不会改写页面。
页面类型不在page\_rewrite\_content\_type的文档不改写。 

### 配置说明:

#### 176. iweb\_page\_rewrite

配置描述：
配置需要进行改写的url。 
配置格式：

	iweb_page_rewrite allow aclname

可分频道配置。 
默认配置：

	none 

#### 177. iweb\_js\_path

配置描述：
配置js文件的路径。
配置格式：

	iweb_js_path pth

可分频道配置。 
默认配置：

	none

#### 178. rtts\_site\_addr
配置描述：
配置m3u8文件所在转码中心域名。 
配置格式：

	rtts_site_addr http://aaa.bbb.ccc 

可分频道配置。 
默认配置：

	none 

#### 179. js\_entry\_func
配置描述：
配置js代码中的入口函数名称。
配置格式：
js_entry_func string 
可分频道配置。
默认配置：
none 

#### 180. web\_service\_url

配置描述：
配置url的替换方式，使用正则表达式。
配置格式：

	web_service_url ^http://www\.m1905\.com/vod/([0-9]+)\.shtml$ http://www.m1905.com/video/getVodinfo.php?id=$1 allow m1905vod 

可分频道配置。 
默认配置：

	none 

#### 181.     player\_div\_id

配置描述：
配置原页面中用于承载flash播放器的div的id值。
配置格式：

	player_div_id m1905_vod_content allow m1905vod 

可分频道配置。 
默认配置：

	none 

配置例子：
	HOST www.m1905.com1 
	acl m1905vod url_regex -i ^http://www\.m1905\.com/vod/[0-9]+\.shtml$ 
	acl m1905video url_regex -i ^http://www\.m1905\.com/video/play/[0-9]+\.shtml
	  
	iweb_page_rewrite allow m1905vod
	iweb_page_rewrite allow m1905video 
	
	iweb_js_path aa/bb/wsIweb.js
	rtts_site_addr http://aaa.bbb.ccc
	js_entry_func m1905_replace 
	
	web_service_url ^http://www\.m1905\.com/vod/([0-9]+)\.shtml$ http://www.m1905.com/video/getVodinfo.php?id=$1 allow m1905vod
	player_div_id m1905_vod_content allow m1905vod 
	
	web_service_url  ^http://www\.m1905\.com/video/play/([0-9]+)\.shtml$ http://www.m1905.com/video/getVideoinfo.php?id=$1 allow m1905video
	player_div_id m1905_video_content allow m1905video 
  
	\#以下三行配置是把对http://www.m1905.com/aa/bb/wsIweb.js的请求转发到我们自己\#搭建的web服务器192.168.21.222上，从而从192.168.21.222上获取wsIweb.js文件， \#同时不会造成js的跨域访问。
	
	acl js_path url_regex -i ^http://www.m1905.com/aa/bb/wsIweb.js$
	cache_peer 192.168.21.222 parent 80 3130 no-query default 
	cache_peer_access 192.168.21.222 allow js_path
	HOST END 

以上用例中 访问http://www.m1905.com/vod/409705.shtml添加:

	<script type="text/javascript" src="aa/bb/wsIweb.js"></script>
	<script language="javascript"> 
	var url = "http://www.m1905.com/video/getVodinfo.php?id=409705"
	m1905_replace(url, "m1905_vod_content", http://aaa.bbb.ccc)
	</script> 

访问http://www.m1905.com/video/play/411120.shtml添加:

	<script type="text/javascript" src="aa/bb/wsIweb.js"></script>
	<script language="javascript"> 
	var url = "http://www.m1905.com/video/getVideoinfo.php?id=411120"; 
	m1905_replace(url, "m1905_video_content", http://aaa.bbb.ccc)
	</script>

对于其他不符合acl规则的页面不进行改写，对于文档类型不为text/html的页面不进行改写。 

## 1.29. 获取服务器端IP

### 功能描述:

对满足特定配置的URL，返回特定的字符串，字符串内容包含服务器用于连接客户端的IP。 

### 相关配置:

	182.  send_server_ip_str 
	183.  get_server_ip_url 

### 配置说明：

#### 182. send\_server\_ip\_str

配置描述：
配置获取IP的URL的格式。 
配置格式：

	get_server_ip_url allow/deny aclname 

可分频道配置。 

默认配置：

	none

#### 183. get\_server\_ip\_url
配置描述：
配置对上述URL的响应内容的格式，在配置的字符串中使用%V来指代字符串。 
配置格式：

	send_server_ip_str string 

可分频道配置。 
默认配置：

	none 

配置例子：

	HOST aaa.aa.com
	acl get_ip url_regex -i ^http://([^/]+)/ip$
	get_server_ip_url allow get_ip 
	send_server_ip_str var server_ip = '%V'; 
	HOST END

请求http://aaa.aa.com/ip时，设squid与客户端连接的IP为1.2.3.4，返回200响应，响应内容为var server\_ip = '1.2.3.4'; 

## 1.30. HTTPS请求

### 功能描述:

针对https请求相关的功能说明。 

### 相关配置:
	
	184.  ssl_session_reuse 

### 配置说明：

#### 184.     ssl\_session\_reuse
配置描述:
对于https请求，squid支持https请求的连接复用，对cache\_peer配置方式的回源支持ssl的session复用，其他情况下没有session复用。比如使用动态源的情况下，squid是没有session复用的。

该配置开启时，可以对没有使用cache\_peer回源方式，使用session复用。 
配置格式:

	ssl_session_reuse on|off 

默认配置:

	off

## 1.31. 授权

### 功能描述：

HTTP请求有时候需要用户认证，HTTP协议采用“盘问-凭据（challenge-credentials）”的方式定义HTTP认证框架。客户端发送请求，如果请求需要认证，服务端发出盘问（challenge），客户端发送凭据（credentials），如果服务端认证通过，返回正常响应，否则，服务端继续发出盘问。由于HTTP会话过程中可能包含代理，代理和源端的认证过程大致相同，红色部分为不同点：
源端认证过程：客户端发送HTTP请求；如果请求需要认证，源端发送401响应，响应中包含一个或多个WWW-Authenticate响应头，该响应头指出源端采用的认证方式（如Basic，Digest等）和相关认证参数；客户端根据WWW-Authenticate响应头的盘问信息和用户名密码，构造认证凭据（credentials），然后发送认证请求，请求中包含Authorization请求头，凭证作为该请求头的值；如果认证通过，返回正常响应，否则，源端继续发送401响应。 

代理认证过程：客户端发送HTTP请求；如果代理需要认证，代理发送407响应，响应中包含一个或多个Proxy-Authenticate响应头，其功能与WWW-Authenticate相同；客户端根据Proxy-Authenticate响应头的盘问信息和用户名密码，构造认证凭据（credentials），然后发送认证请求，请求中包含Proxy-Authorization请求头，其功能与Authorization相同；如果认证通过，可以通过代理继续请求，否则，代理继续发送407响应。 
这里要注意的是WWW-Authenticate和Authorization是End-to-End消息头，Proxy-Authenticate和Proxy-Authorization是Hop-by-Hop消息头。
  
前面简要介绍了HTTP的认证过程，其核心部分是访问认证方式。RFC 2617中定义了两种访问认证方式，分别是Basic和Digest。第三方公司或组织也提供了一些认证方式，如微软提供的Negotiate和NTLM，MIT提供的Kerberos。Squid支持Basic，Digest和NTLM三种认证方式。 
  
Squid作为HTTP代理，正常情况下，须转发WWW-Authenticate和Authorization，不转发Proxy-Authenticate和Proxy-Authorization，我们把这种情况定义成正常行为。但是Squid的某些配置可能打破这种正常行为。这些配置包括：
http\_port的no-connection-auth选项：禁止Squid转发第三方认证方式（NTLM，Negotiate和Kerberos），即如果WWW-Authenticate或Proxy-Authenticate的值是NTLM，Negotiate或Kerberos，将被忽略（见例1）。这个选项的优先级高于cache\_peer的connection-auth选项，因此，它对所有的peer有效。
例1：
 
Squid作如下配置（部分）：

	http_port 3128 vhost vport=80 no-connection-auth 
	cache_peer 192.168.0.3 parent 3128 0 no-query connection-auth=on
	cache_peer_access 192.168.0.3 allow all

如果响应中包含NTLM，Negotiate和Kerberos认证方式，WWW-Authenticate或Proxy-Authenticate响应头将在响应转发过程中被忽略。
cache\_peer的connection-auth，originserver和login选项。它们的描述和例子见cache\_peer。 

以下我们分两种情况分别介绍如何配置Squid使其执行HTTP授权正常行为：
（1）Squid未配置cache\_peer的情况 
为了防止Squid忽略第三方认证方式（NTLM，Negotiate和Kerberos），http\_port不要配置no-connection-auth选项,如例2所示。
例2：
Squid作如下配置（部分）：

	http_port 3128 vhost vport=80


（2）Squid配置cache\_peer的情况 
一般情况下我们很少用到代理认证，因此，为了使Squid符合规范正常行为，我们在cache\_peer中添加connection-auth=on选项，并且http\_port不要配置no-connection-auth选项，目的都是防止Squid忽略第三方认证方式（NTLM，Negotiate和Kerberos）。如例3所示： 
例3：
Squid作如下配置（部分）： 

	http_port 3128 vhost vport=80 
	cache_peer 192.168.0.3 parent 3128 0 no-query connection-auth=on
	cache_peer_access 192.168.0.3 allow all
  
如果cache\_peer配置了originserver选项，须同时配置login=PASS选项（不采用login=PROXYPASS的原因是如果配置了login=PROXYPASS，不存在Authorization，Squid会自动构建Authorization，我们并不希望这种非正常行为产生）。如例4所示：
例4：
 
Squid作如下配置（部分）：
	
	http_port 3128 vhost vport=80 
	cache_peer 192.168.0.3 parent 3128 0 no-query originserver connection-auth=on login=PASS 
	cache_peer_access 192.168.0.3 allow all 

例4要注意的一点是，如果Squid可以直接回源，并无须配置login=PASS选项，因为Squid在peer无法通过的情况下，最后会直接回源，除非配置了never\_direct。 
相关配置：

	cache_peer（见214 cache_peer） 
	http_port（见499 http_port） 
