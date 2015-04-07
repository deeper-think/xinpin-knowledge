---
layout: post
title: 01.squid 请求处理II
category: squid技术
tags: [徐超]
keywords: squid
description:
---

> 用正确的工具，做正确的事情

## 1.11. 重定向

### 功能描述:
用于url改写。

### 相关配置:

	44.   interior_redirector
	45.   interior_redirector
	46.   interior_redirector_with_multimatch
	47.   redirector_rewrite_log_uri
	48.   redirector_rewrite_log_uri_2 
	49.   redirect_url
	50.   uri_replace 
	51.   redirector_access_list1
	52.   redirect_program_list 
	53.   redirector_number 
	54.   redirector_bypass
	55.   url_rewrite_program 
	56.   url_rewrite_children 
	57.   url_rewrite_concurrency
	58.   url_rewrite_host_header
	59.   url_rewrite_access 
	60.   location_rewrite_program
	61.   location_rewrite_children
	62.   location_rewrite_concurrency
	63.   location_rewrite_access

### 配置说明：

#### 44. interior_redirector

配置描述：

用于url的内部重定向。 

    interior_redirector [orig-url] [-i] old_url new_url allow|deny aclname…

orig-url选项表示，回源的时候使用原始的url回源. 
-i选项表示正则表达式匹配的时候忽略大小写。 
old\_url是匹配的请求url，正在表达式匹配。
new\_url是改写后新的url，可以是常规的url或者以301，302状态开始的url，如果配置为301,302状态开头的url，则返回301,302的响应。
该配置与redirect\_url类似，也可以配置返回301,302的响应，该配置比redirect\_url配置功能更为完善，支持使用原url的部分字段作为改写后新url的部分字段。
配置格式:

	interior_redirector [orig-url] [-i] old_url new_url allow|deny aclname… 

可分频道配置。 
默认配置：

	无

配置例子：

	interior_redirector orig-url ^http://[^/:]*(|:[0-9]+)/([^\?]*\?segno=[0-9]+)&key=.*http://ccf.pptv.com/$2 allow url_video pplive_pragma 

#### 45. interior\_redirector\_2

配置描述：
对内部的url重定向，与interior\_redirector\_2配置类似，区别是interior\_redirector在防盗链处理前执行，interior\_redirector\_2在防盗链处理后执行。 
配置格式:

	interior_redirector_2 [orig-url] old_url new_url acl 

可分频道配置。 
默认配置：

	无

#### 46. interior\_redirector\_with\_multimatch
配置描述: 
用于URL的内部重定向。与interior\_redirector和interior\_redirector\_2的区别是：当匹配了某条匹配项后，interior\_redirector和interior\_redirector\_2将停止继续匹配，而本配置项能够进行多条匹配。本配置项在现有内部重定向之前进行。 
配置格式: 
	
	interior_redirector_with_multimatch [orig-url] [-i] old_url new_url allow|deny aclname

可分频道配置 
默认配置: 
	
	无

备注： 
如果HOST域中配置了该配置项，全局域的对其仍然有效。 
配置例子:

	HOST www.fake-host.com
	interior_redirector_with_multimatch .*/fake-page\.html http://www.fake-host.com/fake-page2.html allow all
	interior_redirector_with_multimatch .*/fake-page2\.html http://www.misc-host.com/fake-page3.html allow all 
	interior_redirector_with_multimatch .*/fake-page3\.html http://www.mytest.com:8007/a.html allow all 
	HOST END 

过程:
	1）访问http://www.fake-host.com/fake-page.html，重定向到http://www.mytest.com:8007/a.html;
	2）访问http://www.fake-host.com/fake-page2.html，重定向到http://www.mytest.com:8007/a.html;
	3）访问http://www.fake-host.com/fake-page3.html，重定向到http://www.mytest.com:8007/a.html； 
	4）访问http://www.misc-host.com/fake-page.html，不匹配； 
	5）访问http://www.fake-host.com/misc-fake-page.html，不匹配。 

#### 47. redirector\_rewrite\_log\_uri
配置描述:
与interior\_redirector配置配合使用，用于配置经过interior\_redirector处理后，记录的访问日志url是改写前的还是改写后的，配置为on，则访问日志记录的时候interior\_redirector改写后的url，配置为off或者不配置，则访问日志的url不改变。 
配置格式:

	redirector_rewrite_log_uri on|off

不可分频道配置
默认配置:

	off

#### 48. redirector\_rewrite\_log_uri\_2
配置描述:
与interior\_redirector\_2配置配合使用，用于配置经过interior\_redirector\_2处理后，记录的访问日志url是改写前的还是改写后的，配置为on，则访问日志记录的时候interior\_redirector\_2改写后的url，配置为off或者不配置，则访问日志的url不改变。 
配置格式:

	redirector_rewrite_log_uri_2 on|off 

不可分频道配置 
默认配置:

	off

#### 49. redirect\_url

配置描述:
用于配置请求的url重定向，满足条件的请求，返回301或者302重定向响应，重定向的url为配置的url。如果没有配置301，302状态码，则回源取得重定向后的url响应内容作为请求的响应。
该配置的功能与内部重定向的功能类似，差别是该配置无法实现使用请求url的部分字段作为重定向后url的功能。

配置格式:
	
	redirect_url 301:url allow|deny aclname…* B# m2 ^) M) e! U

可分频道配置
默认配置:

	none 

#### 50. uri\_replace

配置描述：
对uri的某些部分或整体进行替换。 
配置格式：

	uri_replace [part]/regex/string/[flags] allow|deny aclname...

1）part部分为可选部分，指示替换URI的哪个部分。有效值包括s，a，p，f和q，依次表示URI（<scheme>://<authority></path></file><?query>）的5个部分。不配置或无效值默认替换整个URI。part只能是单个字母（性能上的考虑），否则将终止Squid。 
2）regex为perl兼容正则表达式，其中，'\/'表示'/'。
3）string为替换字符串，其中，'\/'表示'/'，'\$'表示'$'；字符串支持$0~$9，实现类似于内部重定向。
4）flags为标识，有效值是'g'和'i'，分别表示全局匹配和忽略大小写；无效值将被忽略。
5）acl的实现类似于内部重定向。 
6）该配置项支持多次匹配，这点类似于interior_redirector_with_multimatch配置项。 
配置格式:

      uri_replace [part]/regex/string/[flags] allow|deny aclname.. 

可分频道配置 
默认配置:
      
	None

配置例子:

	1）s/https/http/ 

将scheme部分的所有“https”替换成“http”。

	2）a/^.*$/127.0.0.1/

将authority整个部分替换成“127.0.0.1”。
 
	3）p/^.*$//

将path整个部分替换成空字符串。

	4）f/_{2}([a-fA-F0-9]{2})/%$1/gi
 
将file部分的类似“__af”的子串替换成“%af”。

	5）q/^.*$//

将query整个部分替换成空字符串。
 
	6）/%([a-fA-F0-9]{2})/__$1/gi 

将整个uri的类似“%af”的子串替换成“__af”。 

	7）z/%([a-fA-F0-9]{2})/__$1/gi 

z为无效值，因此，其效果跟6相同。

#### 51. redirector\_access\_list

配置描述:
未使用

#### 52. redirect\_program\_list

配置描述:
未使用: 

#### 53. redirector\_number

配置描述:
未使用 

#### 54. redirector\_bypass

配置描述:
未使用: 

#### 55. url\_rewrite\_program

配置描述:
未使用

#### 56. url\_rewrite\_children

配置描述:
未使用

#### 57. url\_rewrite\_concurrency

配置描述:
未使用 

#### 58. url\_rewrite\_host\_header

配置描述:
未使用 

#### 59. url\_rewrite\_access

配置描述:
未使用 

#### 60. location\_rewrite\_program

配置描述:
未使用 

#### 61. location\_rewrite\_children

配置描述:
未使用 

#### 62. location\_rewrite\_concurrency

配置描述:
未使用

#### 63. location\_rewrite\_access
配置描述:
未使用 

## 1.12.  range请求处理

### 功能描述:

对range请求的响应处理 

### 相关配置:
	
	64.   range_complex_do 

### 配置说明：

#### 64. range\_complex\_do

配置描述:
配置为on则对多range重叠、乱序返回206部分文件内容，有缓存情况下不回源。配置为off时，兼容以前版本。
配置格式:

	range_complex_do on|off 

可分频道配置 
默认配置:

	off

## 1.13. 去问号处理

### 功能描述:

很多动态页面，响应的内容不会因为问号后面的参数不一样而不一样。可通过该配置，忽略问号后面的参数。 
与内部重定向功能相似，用内部重定向也可以配置达到该目的，这个配置只是更直观而已。 

### 相关配置:
	
	65.   url_cut_question_mark 

### 配置说明:

#### 65. url\_cut\_question\_mark

配置描述：
与url重定向功能一样，用于删除url问号后面的内容. 
配置格式：

	url_cut_question_mark allow|deny aclname … 

可分频道配置。 
默认配置：

	none

配置例子：

	acl allow_cut_question hash_url_regex .ku6.cn -i ^http://.*\.ku6\.cn/.* 
	url_cut_question_mark allow allow_cut_question 

访问如下: 

	pollurl 'http://img1.c0.ku6.cn/flvPlayer.swf?a=1?b=1
	pollurl 'http://img1.c0.ku6.cn/flvPlayer.swf?a=1?b=1?c=1 
	pollurl 'http://img1.c0.ku6.cn/flvPlayer.swf?a=1?b=1?c=1?d=1
	pollurl 'http://img1.c0.ku6.cn/flvPlayer.swf?a=1?b=1?c=1?d=1 

四个请求，均按http://img1.c0.ku6.cn/flvPlayer.swf重定向，故仅缓存一份。 

## 1.14. 缓存回源验证

### 功能描述:

拒绝对刚撤单、但域名没有切换的客户的服务，从而减少此类的流量。 

### 相关配置:

	66.   checkclientdns 

### 配置说明:

#### 66. checkclientdns
配置描述:
由于squid在HIT处理时，直接从squid响应请求，而不回源，这个过程不会检查该请求域名的DNS，而此请求，可能是对一个刚撤单、但域名没有切换的域名的请求，所以该配置项，功能为在HIT处理时，是否对该域名进行DNS检查，如果NDS检查此域名没有IP，则不会对该请求服务。该处理过程不会延迟服务（由于异步实现）。 
配置为on则开启这功能，不配置或者配置为off则关闭该功能。
配置格式:

	checkclientdns on|off 
   
不可分频道 

默认配置：

	off

## 1.15. 防攻击

### 功能描述:

限制客户IP在一段时间内允许访问的次数，这里的客户IP：若存在Cdn-Src-Ip头，则以其内容作为客户IP，否则，取请求中的client_addr作为客户IP。 

### 相关配置:

	67.   catchtime_len
	68.   access_maxtimes 
	69.   access_miss_maxtimes
	70.   no_do_times_attack
	71.   timeattack_deny_len 
	72.   total_rate_limit 
	73.   miss_rate_limit
	74.   catch_rate_time
	75.   attack_rate_limit 
	76.   attack_miss_limit 
	77.   deny_attack_time 
	78.   denyattack_method
	79.   clean_access_table_ival 
	80.   clean_access_table_nums 
	81.   access_log增加log_attack选项
	82.   attack_session 
	83.   denyattack_redirect
	84.   attack_alarm_log
	85.   antiattack_strategy 

### 配置说明：

#### 67. catchtime\_len

配置描述：
配置统计访问次数的时间长度，以秒为单位。 
配置格式：

	catchtime_len N

可分频道配置。
默认配置：

	1 

#### 68. access\_maxtimes

配置描述：
配置在catchtime\_len时间内，允许客户IP访问的次数
配置格式：

	access_maxtimes N 

可分频道配置。 
默认配置：

	0，即默认情况下不做防攻击检测。 

备注： 
动态配置的次数，在squid重启或者reconfigure后失效。

#### 69. access\_miss\_maxtimes

配置描述：
配置在catchtime\_len时间内，允许单个客户访问且为TCP_MISS的次数。
配置格式：

	access_miss_maxtimes N 

可分频道配置。 
默认配置：

	0，即默认情况下不做防攻击检测。 

备注： 
动态配置的次数，在squid重启或者reconfigure后失效。 
配置例子：

	catchtime_len 60 
	HOST www.test.com 
	access_maxtimes 20 
	access_miss_maxtimes 9 
	HOST END 

则：如果某个客户IP在60秒钟内，访问www.test.com的次数超过20次，或者得到TCP_MISS超过9次，将会被判定为攻击。 

#### 70. no\_do\_times\_attack

配置描述：
配置对某些域名，或者某些客户端不做次数攻击的检测 
配置格式：

	no_do_times_attack deny/allow aclname 

可分频道配置。
默认配置：

none

配置例子：

    acl No_Attack_IP cdnsrc 127.0.0.1
	no_do_times_attack deny !No_Attack_IP

#### 71. timeattack\_deny\_len

配置描述：
当发现某个ip的请求属于次数攻击时，把此ip放入黑名单中，在之后的一段时间timeattack\_deny\_len内，对此ip的请求拒绝访问。在timeattack\_deny\_len时间后，把此ip从黑名单中删除，正常对其提供服务。

配置格式：

	timeattack_deny_len 30 minutes 

可分频道配置。 
默认配置：

	none 

#### 72. total\_rate\_limit

配置描述：
配置每秒钟某个域名的总流量限制值。当总流量大于限制值时，即认为其受到攻击。对于受到攻击的域名，从下一个请求开始进行防攻击处理。 
配置格式：
    
	total_rate_limit ..KB

可分频道配置。 
默认配置：

	none

配置例子：

	total_rate_limit 20 KB 

#### 73. miss\_rate\_limit

配置描述：
配置每秒钟某个域名的MISS流量限制值。当MISS流量大于限制值时，即认为其受到攻击。对于受到攻击的域名，从下一个请求开始进行防攻击处理。
配置格式：

	miss_rate_limit .. KB

可分频道配置。
默认配置：

	none    

配置例子：
	
	miss_rate_limit 10 KB

#### 74. catch\_rate\_time

配置描述：
计算在这个时间长度内，某个域名每秒钟的平均流量，当其超过total\_rate\_limit或者miss\_rate\_limit的限制时，认为此域名受到攻击。此配置可以带单位，没有单位时默认以秒为单位。
配置格式：

	catch_rate_time time_t 

可分频道配置。 
默认配置：

	30秒 

配置例子：

	catch_rate_time 1 minutes 

#### 75. attack\_rate\_limit

配置描述：
配置防攻击时，每秒钟某个域名的总流量限制值。 
如果配置了denyattack_method方式4，但没有配置attack_rate_limit，则没有防攻击。
配置格式：

	attack_rate_limit 20 KB

可分频道配置。
默认配置：

	none 

#### 76. attack\_miss\_limit

配置描述：
配置防攻击时，每秒钟某个域名的MISS限制值。
配置格式：

	attack_miss_limit 20 KB 

可分频道配置。 
默认配置：

	none

#### 77. deny\_attack\_time

配置描述：
配置由攻击态回到正常态的时间长度。以分钟为单位。 
当判定某个域名受到攻击后，此后的一段时间，均进行防攻击（比如：限速）。直到deny\_attack\_time时间后，才回到正常态（比如：不限速）。

配置格式：

	deny_attack_time N

不可分频道配置。 
默认配置：

	30 

#### 78. denyattack\_method

配置描述：
配置当请求判定为攻击时的处理方法。 
根据访问次数判定为攻击时有3种处理方法，根据访问流量判定为攻击时有4种处理方法。 
以下为根据访问次数判定为攻击时的3种处理方法：

为1：则在日志中记录Warning 
为2：则返回403错误给客户端
为3：则不返回任何内容 
为4：进行限流量 

	denyattack_method N1:N2
	N1 as: 
    1: ACCESS_TIMES_TOATL 
    2: ACCESS_TIMES_MISS 
    3: RATE_TOTAL 
    4: RATE_MISS

	N2 as:
	1: Log in access_log 
	2: return 403
	3: return NULL content 
	4: slow rate(only for rate_attack)

配置格式：

	denyattack_method N1:N2 

可分频道配置。 
默认配置：

	无 

#### 79. clean\_access\_table\_ival

配置描述:
无用配置 

#### 80. clean\_access\_table\_nums

配置描述:
无用配置

#### 81. access\_log增加log\_attack选项

配置描述：
配置log文件只记录受攻击时的请求。
日志格式%aty %adm %aat %amt %aar %amr 
配置格式：

    log_attack=on|off

不可分频道配置。
默认配置：

	无 

备注：

	log_attack=on选项的位置，必须在配置的文件路径的后面。 

配置例子：
例1:

	logformat combined_attack [%tl] "%rm %ru HTTP/%rv" %Hs %<st %Ss:%Sh %aty %adm %aat %amt %aar %amr 
	access_log /usr/local/squid/var/logs/attack.log log_attack=on combined_attack
	%aty记录攻击的类型，有4个类型：
	ACCESS_TIMES_TOATL，总访问次数过多的攻击
	ACCESS_TIMES_MISS，MISS访问次数过多的攻击 
	RATE_TOTAL，总流量过高的攻击
	RATE_MISS，MISS流量过高的攻击
	%adm记录防攻击的方式，有4个：WARNING，RET_403，RET_NULL，SLOW_RATE分别对应denyattack_method配置的4个方式 
	%aat记录总的访问次数
	%amt记录MISS访问次数
	%aar记录总的流量 
	%amr记录MISS的流量 

注意：这里记录的流量是当前这一秒钟的流量，如果有个文件，前面两秒已经发送了20K，第三秒发送剩下的50字节，则，记录的流量为50字节。
例2：
用例描述：某个ip短暂的攻击。

配置：

	catchtime_len 60 
	access_maxtimes 50 
	denyattack_method 1:2
	timeattack_deny_len 30 minutes 

某个ip在某一分钟，突发的请求了100个请求，则前50个请求正常响应，后50个请求返回403。在之后的30分钟内，ip存在于黑名单中，对于此ip的请求返回403。如果这期间，ip没有再频繁访问，则30分钟后，恢复对此ip的正常服务，把其从黑名单中删除。如果仍处于攻击，则继续放入黑名单中。

#### 82. attack\_session

配置描述：
标识用户session的id名称，此配置中配置的所有id在用户cookie中必须全部存在，否则视为没有cookie。 
配置的session的id将与IP一起参与key的计算。
配置格式：

	attack\_session id1 id2 

可分频道配置。 
默认配置：

	none
 
#### 83. denyattack\_redirect

配置描述：
当判定请求为攻击时，跳转到验证码页面。
配置格式：

	denyattack_redirect old_url 301:newurl allow/deny aclname，返回301响应

可分频道配置 
默认配置：

	None 

#### 84. attack\_alarm\_log

配置描述：
用于配置当发生访问次数或者流量攻击的时候，记录日志的路径。报警脚本会定时处理该文件，将报警上报给wsms服务器。

配置格式:

	attack_alarm_log path 

默认配置:

	None

#### 85. antiattack\_strategy

配置描述:
为攻击类型配置防攻击策略。
配置格式:

	antiattack_strategy attack_type_name strategy_name strategy_argv 
	antiattack_strategy attack_type_name WARNING 
	antiattack_strategy attack_type_name RET_403 
	antiattack_strategy attack_type_name RET_NULL 
	antiattack_strategy RATE_TOTAL|RATE_MISS SLOW_RATE rate_limit_kb 
	antiattack_strategy ACCESS_TIMES_TOTAL|ACCESS_TIMES_MISS REDIRECT_FOR_TIMES [-i] old_url new_url allow aclname... 
	antiattack_strategy attack_type_name REDIRECT [-i] old_url new_url allow aclname... 
	 
	attack_type_name的固定取值有： 
	ACCESS_TIMES_TOTAL：表示总访问次数攻击 
	ACCESS_TIMES_MISS：表示MISS次数攻击 
	RATE_TOTAL：表示总流量攻击 
	RATE_MISS：表示MISS流量攻击 
	BW_CONTROL：表示带宽控制 
	但attack_type_name可以使用自定义名称，主要用于SQL注入攻击。 
  
	strategy_name的有效取值有： 
	WARNING：表示警告 
	RET_403：表示返回403响应 
	RET_NULL：表示直接关闭连接 
	SLOW_RATE：表示限制流量，只用于流量攻击 
	REDIRECT_FOR_TIMES：表示次数攻击的重定向，只用于次数攻击 
	REDIRECT：表示重定向 
	strategy_name不可以自定义名称，除非新增新的策略，否则，结束进程。 

默认配置:

	None 

配置例子:
例1：流量攻击 

	HOST aaa.aa.com 
	total_rate_limit 20 KB 
	miss_rate_limit 10 KB 
	catch_rate_time 1 minutes 
	\# denyattack_method 3:1 
	antiattack_strategy RATE_TOTAL WARNING 
	\# denyattack_method 4:2 
	antiattack_strategy RATE_MISS RET_403 
	HOST END 
	HOST ddd.dd.com 
	total_rate_limit 20 KB 
	miss_rate_limit 10 KB 
	catch_rate_time 1 minutes 
	\# denyattack_method 3:3 
	antiattack_strategy RATE_TOTAL RET_NULL 
	\# denyattack_method 4:4 
	\# attack_miss_limit 10 KB 
	antiattack_strategy RATE_MISS SLOW_RATE 10 
	HOST END 

请求aaa.aa.com，如果1分钟内的总流量超过1200KB，攻击告警 ;
请求aaa.aa.com，如果1分钟内的MISS流量超过600KB，返回403响应; 
请求ddd.dd.com，如果1分钟内的总流量超过1200KB，直接关闭链接； 
请求ddd.dd.com，如果1分钟内的MISS流量超过600KB，降低MISS流量的限制值为600KB。 

例2：次数攻击 

	HOST eee.ee.com 
	access_maxtimes 30 
	access_miss_maxtimes 10 
	catchtime_len 60 
	\# denyattack_method 1:1 
	antiattack_strategy ACCESS_TIMES_TOTAL WARNING 
	\# denyattack_method 2:25  
	antiattack_strategy ACCESS_TIMES_MISS RET_403 
	HOST END 
	HOST bbb.bb.com 
	access_maxtimes 30 
	access_miss_maxtimes 10 
	catchtime_len 60 
	\# denyattack_method 1:3 
	antiattack_strategy ACCESS_TIMES_TOTAL RET_NULL 
	\# denyattack_method 2:5 
	\# denyattack_redirect ^http://[^/]+/(.*) http://ddd.dd.com/verify.php allow all  
	antiattack_strategy ACCESS_TIMES_MISS REDIRECT_FOR_TIMES ^http://[^/]+/(.*) http://ddd.dd.com/verify.php allow all 
	HOST END 

请求eee.ee.com，如果1分钟内的总访问次数超过30次，攻击警告； 
请求eee.ee.com，如果1分钟内的MISS次数超过10次，返回403响应； 
请求bbb.bb.com，如果1分钟内的总访问次数超过30次，直接关闭链接；
请求bbb.bb.com，如果1分钟内的MISS次数超过10次，重定向到验证页面。 

例3：带宽控制

	HOST ccc.cc.com
	\# bwctrl_deny_method 28 
	\# deny_redirector ^http://[^/]+/(.*) http://ddd.dd.com/$1 allow all
	antiattack_strategy BW_CONTROL REDIRECT ^http://[^/]+/(.*) http://ddd.dd.com/$1 allow all
	HOST END
	HOST fff.ff.com 
	\# bwctrl_deny_method 3 
	antiattack_strategy BW_CONTROL RET_NULL 
	HOST END 

请求ccc.cc.com，如果超出带宽控制限制值，直接重定向； 
请求fff.ff.com，如果超出带宽控制限制值，直接关闭链接。 

## 1.16. set-cookie防攻击

### 功能描述:

采用set-cookie的方式进行防攻击。采用set-cookie的方式进行防攻击，针对该功能了log_format新增可选字段%sc，响应中不增加set-cookie，则打印NONE，响应中增加set-cookie，如果是cookie错误则打印CK\_ERR， 如果是cookie过期则打印CK\_EXP， 如果是cookie为空则打印CK\_NULL，set\_cookie功能关闭，则打印- 
  
使用例子如下(具体配置使用可见配置说明):
例如： 
	
	HOST aaa.aa.com
	acl test url_regex index.html
	set_cookie_check_access allow all 
	set_cookie_home_page_return allow test
	set_cookie_encrypt_style $ourkey$remote_addr:wskey:WSKEY:-:60
	set_cookie_encrypt_req_style $key$time 
	set_cookie_encrypt_time_type %s%x
	HOST END 
  
则根据如上配置，假设客户端ip为127.0.0.1:

加密串为wskey,加密的内容为：wskey127.0.0.1 

采用md5加密后值为：printf "%s" wskey127.0.0.1|md5sum， 为d6251dce4005c59ad58d9b148322dd8e5 

当前时间十六进制显示为：printf "%x" `date +%s`，值为503adeae 
由于配置格式为$key$time,所以最后cookie的<key=value>内容为：WSKEY=d6251dce4005c59ad58d9b148322dd8e503adeae 
过期时间为：60秒。
当客户端访问时，以上面配置为例
如果访问为首页，即以index.html结尾的页面：
1、头部没有cookie，cookie的内容<key=value>中的key不为WSKEY或者value验证错误时（包括加密错误或者时间过期）
则正常响应，但是响应头中要加上 set-cookie:WSKEY=d6251dce4005c59ad58d9b148322dd8e503adeae内容 ,如果需要回源，则回源应去掉cookie头内容 .
2、如果头部cookie验证正确，则正常响应,如果需要回源，则回源应去掉cookie头内容。
  
如果访问为非首页，即不以index.html结尾的页面： 
1、头部没有cookie，cookie的内容<key=value>中的key不为WSKEY或者value验证错误时（包括加密错误或者时间过期） 
则直接返回302响应，且在响应头中要加上set-cookie:WSKEY=d6251dce4005c59ad58d9b148322dd8e503adeae内容
2、如果头部验证正确，则正常响应
如果需要回源，则回源应去掉cookie头内容

### 相关配置:

	86.   set_cookie_check_access 
	87.   set_cookie_home_page_return
	88.   set_cookie_encrypt_style
	89.   set_cookie_encrypt_req_style 
	90.   set_cookie_encrypt_time_type 

### 配置说明:

####　86. set\_cookie\_check\_access

配置描述：
是否允许使用set-cookie功能
配置格式：

	set_cookie_check_acces allow/deny aclname

可分频道配置 
默认配置:

	none, 等同deny all 

#### 87. set\_cookie\_home\_pag\e_return

配置描述：
配置首页的响应策略，allow为正常响应，deny为返回302. 
配置格式：

	set_cookie_home_page_return allow/deny aclname 

可分频道配置 
默认配置:

	none,等同于deny all，返回302 

#### 88. set\_cookie\_encrypt\_style

配置描述：
配置cookie相关的内容(配置方法与encrypt\_style相同，具体可察看该配置），包括加密串，加密内容，过期时间，cookie对<key=value>中key的名称。

set\_cookie\_encrypt\_style field1:field2:field3:field4:field5
field1:配置欲加密的字符串的生成方式，如：$ourkey$remote_addr) 
field2:配置生成cookie的加密串（由于以：为分隔符，加密串不能包含：）
field3:配置cookie的key名称 
field4:配置时间串的名称（本功能不用该字段，配置为-)) 
field5:cookie过期时间，单位为秒 

field1子配置说明：

	$remote_addr: client_addr 
	$ourkey: the key we set
	$time: the time stamp in request 

还有其他可选值，具体可查看encrypt\_style配置的介绍，本功能只用到以上三个. 
配置格式:

	set_cookie_encrypt_style field1:field2:field3:field4:field5

可分频道配置
默认配置:

	none 

配置例子:

	set_cookie_encrypt_style $ourkey$remote_addr:wskey:WSKEY:-:60

即 key=WSKEY
   md5<<wskey><$remote_addr>> 
过期时间60秒，其中对于本set-cookie功能，$ourkey$remote\_addr为固定配置。
 
#### 89. set\_cookie\_encrypt\_req\_style

配置描述:
配置cookie对<key=value>中value的格式(配置方法与encrypt\_req\_style 相同，具体可察看该配置） 
	
	set\_cookie\_encrypt\_req\_style $key$time
	$key ：根据set_cookie_encrypt_style配置中field1得出的md5值3 @1 H5 k$ [4 l; L" |; Y! l# n* r
	$time：根据set_cookie_encrypt_time_type配置得出的时间串" H' q0 c; s9 o. W) _/ i# A

配置格式:
	set_cookie_encrypt_req_style $key$time

可分频道配置 

默认配置：

	set_cookie_encrypt_req_style $key$time 

#### 90. set\_cookie\_encrypt\_time\_type

配置描述：
配置时间的格式(配置方法与encrypt\_time\_type 相同，具体可察看该配置） 
配置格式：

	set_cookie_encrypt_time_type %s%x 	
可分频道配置 
默认配置:

	set_cookie_encrypt_time_type %s%x

配置例子:
	
	set_cookie_encrypt_time_type %s%x 

其中%x表示十六进制显示，%s为从`00:00:00 1970-01-01 UTC'到当前时间的秒数

## 1.17. 防盗链

### 功能描述:

防盗链包括以下几种: 
17173防盗链, Sina防盗链, QQ验证库防盗链(腾讯长尾流媒体)，回源添加防盗链串功能，通用防盗链（通用防盗链还包含客户特殊需求的防盗链，比如pplive防盗链，百度音乐，QQ视频，土豆，彩云,sohu xxtea防盗链,天翼视讯防盗链）。
Squid实现的功能： 
	1．验证url的时间和md5(key值)，如果过期或key值不符，则不允许访问(403).
	2．对(域名，文件名)相同，而(时间，md5)不同的url，实际访问的是相同的文件，squid可以正确缓存。(如：缓存了http://down.iask.com/20071116095 ... 9537fcd39/12345.flv，则对请求http://down.iask.com/20071116102 ... 9363c3c40/12345.flv也能命中，并返回前一个url的内容) 
	query_access_url配置的防盗链请求格式通常是: 
	域名/过期时间/md5验证码/实际文件名
	如http://down.iask.com/20071113103 ... 4b3a6c555/12345.flv 
	如果时间过期或md5验证码不正确则不允许访问；
	过期时间保证一个url只在一段时间内有效；
	md5由一个干扰串，时间，和文件名计算出来，保证客户无法自己构造url 

各种防盗链实现方式如下: 
Sohu xxtea防盗链：
	1) 从请求的url中取出key
	2) 对key进行字符串替换，得到字符串A，替换规则包括：
		-替换成+ 
		_替换成/! 
		.替换成=! 
	3) 将字符串A进行base64的解码，生成字符串B 
	4) 使用字符串B和加密串xxteakey做xxtea的解码，得到字符串C) O, o1 a+ W2 `1 ?
	5) 对字符串C进行分析： 
		a）以'|'为分隔符，前半段是时间串timestr，后半段parturl。如下格式: 
		当前时间的分钟数|文件名的前8位。
		b）timestr表示的是key生成时的当前时间的分钟数，如果与解密时的时间相差一个小时以上，则防盗链验证失败。 
     	parturl与文件名的前8个字符如果不一致，则防盗链验证失败。
		如果不足8个字符，则有几个字符就比较几个。

注：
1. 文件名为去掉路径部分 
2. 用于xxtea算法解密的key为!#^&cDnvideo
3. 该例子中的key解码后为21247655|703eec26 
检查解密后的字符串，需要做如下匹配： 
1. 当前时间的分钟数必须在当前标准时间前后1小时的范围内。 
2. 文件名前8伟必须跟url里面的访问文件名（url路径去掉目录）前8位一致。
如果匹配，允许访问。否则返回403，禁止访问。

彩云防盗链:
现有客户开心听对http点播加速的域名有特殊的防盗链需求，主要有以下三个方面： 1 BASE64 decode 解码
2 异或运算 
3 时间戳防盗链
烦请审核，解密过程中涉及的
BASE64 decode
和
异或运算
部分当前平台是否可以另外配置实现。 
附上整个加密和解密详情： 
一、加密过程 
1、 加密前url 
http://domain/xxx.yy
domain是域名
其中xxx.mp3 的格式举例如下： /11/32/10000022\_2629.mp3 
2、 获取当前时间
MMDDHHmm
MM:月份
DD:天数 
HH:小时
mm:分钟
例如： 02070408 为 2月7号4点08分
3、 将第一步的xxx和第二部的时间串用-号连接 
xxx-MMDDHHmm 
如前例为 
/11/32/10000022_2629-02070408 
4、 对第3部得到的字符串中的数字字符和0x01做异或
例如第三步中的字符串将变为/00/23/01111133_3738-131615197 
5、 对第四步得到的字符串做标准BASE64 encode 得到新字符串 
例如上一个字符串得到LzAwLzIzLzAxMTExMTMzXzM3MzgtMTQwNzA1MTQ= 
（可以不是十分准确，只是举例）
6、 在第五步得到的字符串前面加上\/duomial\/ 后面加上最开始的.
例如得到字符串 
/duomial/LzAwLzIzLzAxMTExMTMzXzM3MzgtMTQwNzA1MTQ=.mp3 

二、解密验证过程： 
1、输入的字符串为加密得到的字符串加域名 例如： 
http:// domain/duomial/LzAwLzIzLzAxMTExMTMzXzM3MzgtMTQwNzA1MTQ=.mp3
取出/duomial/LzAwLzIzLzAxMTExMTMzXzM3MzgtMTQwNzA1MTQ=.mp3 
2.验证是第1步的字符串种否有/duomial/，没有报错。 
3．去掉/duomial/和后缀，得到LzAwLzIzLzAxMTExMTMzXzM3MzgtMTQwNzA1MTQ= 
4．对第三部得到的字符串做BASE64 decode 得到/00/23/01111133\_3738-13161519 
5. 对第四步得到的字符串中的数字字符和0x01做异或得到
/11/32/10000022\_2629-020704083
6.取出时间戳02070408 获得当前时间戳 验证时间是否过期
过期验证标准为取出时间戳之前5分钟到之后30分钟 
7．加上后缀得到真实链接 
/11/32/10000022\_2629.mp3 


## 1.18. QQ音乐验证

### 功能描述:

QQ音乐验证的流程：
1. 请求头中有白名单的Referer头，
有cookie头，则进行Referer验证（与白名单中对应的qqmusic_fromtag是否匹配）。 
没有cookie头，则进行防盗链串的验证。
2. 请求头中没有Referer头或者对应的Referer头没有在白名单中， 
有cookie头，则进行lib库验证。 
没有cookie头，则进行防盗链串的验证。
3. 三种验证不通过，则返回403，拒绝访问。 
4. 进行Referer验证或者lib库验证通过，如果qqmusic\_redirect打开而且请求中带Cdn-Src-Ip，则返回302重定向，在Location里面加防盗链串。否则，继续处理请求。

### 相关配置:

	130.  qqmusic_access
	131.  qqmusic_access_pass 
	132.  cookie_to_url
	133.  qqmusic_redirect

### 配置说明:

#### 130.     qqmusic\_access

###　配置描述：

用于指定是否进行QQ音乐lib库验证。
配置格式：

	qqmusic_access deny|allow aclname… 

可分频道配置。 
默认配置：
无

#### 131.     qqmusic\_access\_pass

配置描述：
用于指定白名单，使用Referer验证，不进行lib库验证。 
配置格式：

	qqmusic_access_pass deny/allow aclname

可分频道配置。
默认配置：

	none 

#### 132.     cookie\_to\_url
配置描述：
用于配置，QQ验证要重定向返回时，将Cookie头中的一些参数信息记录到url中。
配置格式：

	cookie_to_url wordlist

可分频道配置。
默认配置：

	none 

备注：
如果配置多个，用空格隔开。
配置例子：
这个配置最好有个例子比较清晰

#### 133.     qqmusic\_redirect

配置描述：
开关配置，当其打开而且请求中带Cdn-Src-Ip，对于验证成功(Referer验证或者库验证)的进行302重定向，当其关闭时，则不进行重定向。 
配置格式：

	qqmusic_redirect on/off 

可分频道配置。
默认配置：

	off，即对验证通过的，不进行302重定向。

配置例子：
例如： 
QQ白名单：$stream1.qqmusic.qq.com 13469 
则squid配置：

	acl qqmusic_ref req_header Referer ^(http://|)stream1\.qqmusic\.qq\.com 
	acl qqmusic_cook req_header Cookie qqmusic_fromtag=(0|1|3|4|6|9)($|;|\s) 
	qqmusic_access_pass allow qqmusic_ref qqmusic_cook 
	qqmusic_access deny qqmusic_ref 
	forward_key_name key
	forward_time_name t 
	forward_time_expire 25 seconds 
	forward_key_secret letv_password
	md5_style ourkey$uri$time
	remove_iphost on 

如果请求头没有Referer或者Referer头不匹配stream1.qqmusic.qq.com，进行库验证。

	pollurl -p 8080 http://www.testa.com/ --header="Cookie: qqmusic_guid=0CA78319C9DF481B80B8FC33AFD61A59; qqmusic_gkey=5BA486E27F7140897D14F84C52940AE334E93AD133794D5B; qqmusic_fromtag=0" 经过QQ验证库验证，允许访问。
	pollurl -p 8080 http://www.testa.com/ --header="Cookie: qqmusic_guid=0CA78319C9DF481B80B8FC33AFD61A59; qqmusic_gkey=5BA486E27F7140897D14F84C52940AE334E93AD133794D5B; qqmusic_fromtag=0\nCdn-Src-Ip:192.168.21.10" 

返回302重定向，http://127.0.0.1/www.testa.com/? ... 38&t=1268369379。

	pollurl -p 8080 http://www.testa.com/ --header="Cookie: qqmusic_guid=0CA78319C9DF481B80B8FC33AFD61A59; qqmusic_gkey=5BA486E27F7140897D14F84C52940AE334E93AD133794D5B; qqmusic_fromtag=0\nReferer:www.kk.com"经过QQ验证库验证，允许访问。 
 
如果请求头有Referer，而且Cookie中的qqmusic_fromtag为0,1,3,4,6,9其中的一个，则允许访问，不必经过库验证。 

	pollurl -p 8080 http://www.testa.com/ --header="Cookie: qqmusic_guid=0CA78319C9DF481B80B8FC33AFD61A59; qqmusic_gkey=5BA486E27F7140897D14F84C52940AE334E93AD133794D5B; qqmusic_fromtag=9\nReferer:stream1.qqmusic.qq.com" 没经过库验证，允许访问。 
  
	pollurl -p 8080 http://www.testa.com/ --header="Cookie: qqmusic_guid=0CA78319C9DF481B80B8FC33AFD61A59; qqmusic_gkey=5BA486E27F7140897D14F84C52940AE334E93AD133794D5B; qqmusic_fromtag=9\nReferer:stream1.qqmusic.qq.com\nCdn-Src-Ip:192.168.21.10" 
返回302重定向，Location: http://127.0.0.1/www.testa.com/? ... b1&t=1268377440
  
再次访问pollurl -p 8080 http://127.0.0.1/www.testa.com/? ... b1&t=1268377440 
允许访问。 
  
如果请求头有Referer，但是没有Cookie头，或者Cookie头没有qqmusic\_fromtag的key值，或者Cookie中的qqmusic\_fromtag与白名单不匹配，则拒绝访问。

	pollurl -p 8080 http://www.testa.com/ --header="Referer:stream1.qqmusic.qq.com" 
	pollurl -p 8080 http://www.testa.com/ --header="Cookie: qqmusic_guid=0CA78319C9DF481B80B8FC33AFD61A59; qqmusic_gkey=5BA486E27F7140897D14F84C52940AE334E93AD133794D5B;\nReferer:stream1.qqmusic.qq.com"
	pollurl -p 8080 http://www.testa.com/ --header="Cookie: qqmusic_guid=0CA78319C9DF481B80B8FC33AFD61A59; qqmusic_gkey=5BA486E27F7140897D14F84C52940AE334E93AD133794D5B; qqmusic_fromtag=2\nReferer:stream1.qqmusic.qq.com" 
  
	请求既没有Referer，也没有Cookie，则拒绝访问。
	pollurl -p 8080 http://www.testa.com/




## 1.19. 网易回源验证

无用配置

## 1.20. Nokia源授权方案

无用配置