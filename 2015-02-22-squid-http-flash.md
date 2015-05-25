---
layout: post
title: 02.squid HTTP流媒体
category: squid技术
tags: [徐超]
keywords: squid
description:
---

> 用正确的工作，做正确的事情

## 2.1. 流媒体快速启动

### 功能描述:

流媒体（比如在线视频等），在启动的时候，速度不快；而用户一般都希望在观看视频文件的时候，减少缓冲时间，能够快速启动起来；为了满足用户的这个需求，需要对流媒体文件进行启动加速。 
该功能的实现大致如下： 
squid做hit处理时，把流媒体文件的前一部分（根据缓冲容量），缓冲在内存中。如果请求过来，那么先把内存中缓存的部分发送给客户端，加快了流媒体的缓冲速度，进而加快启动速度；流媒体启动后，剩余的文件内容从磁盘I/O获取。
附加说明：
1. 内存有限，最好配置一些特定类型的文件进行缓存。 
2. 目前该功能已经从图片产品线的应用上删除了。 

### 相关配置:

185.  cache\_disk\_mem 
186.  hotspot\_access 
187.  hotspot\_bitrate 
188.  hotspot\_time 
189.  disk\_memory\_replacement\_policy 
190.  hotspot\_per\_cache 

### 配置说明：

#### 185. cache\_disk\_mem

配置描述：
总的快速启动内存大小，全局配置 
配置格式：

	cache_disk_mem bytes 

不可分频道配置。 
默认配置：
	
	8MB 

#### 186. hotspot\_access
 
配置描述：
热点文件配置，比如可按文件类型，url规则等。 
配置格式：

	hotspot_access allow|deny aclname… 

可分频道配置。 
默认配置：

	none，所有文件都不为热点

#### 187. hotspot\_bitrate

配置描述：
视频文件码率配置 
配置格式：

	hotspot_bitrate bytes

可分频道配置。
默认配置：

	50K

#### 188. hotspot\_time 
配置描述：
热点视频文件缓冲时间.
每个请求在快启缓存中可分配的最大空间是hotspot_bitrate* hotspot_time，因此默认是300k。

配置格式：

	hotspot_time time 

可分频道配置。
默认配置：

	6

#### 189. disk\_memory\_replacement\_policy
配置描述：
disk_mem替换策略 
配置格式：

	disk_memory_replacement_policy lru 

可分频道配置。
默认配置：

	lru 

备注： 
测试的时候可以不配该配置。替换策略当前版本只支持lru。 

#### 190. hotspot\_per\_cache

配置描述:
每个热点文件缓存内存的大小，该配置值=hotspot\_bitrate* hotspot\_time
配置格式:

	hotspot_per_cache bytes 

可分频道配置 
默认配置:

	300 KB

## 2.2. 视频拖拉

### 功能描述:

支持mp4,flv等视频文件的时间，字节视拖拉，同时可配置支持使用nginx的MP4时间按拖拉。
对pptv客户支持配置采用pptv的拖拉算法。 
对于56用户的拖拉行为进行特殊处理：
背景： 
对于56用户的特殊处理方式，总体上按照客户给出的文档进行开发，从文档中这里出以下需求（按照整体处理流程）： 
示例：http://xxxx/xxx.flv?m=s&start=123&end=456&h=1&h1=123123 
1、当m不存在或m !=s 并且m != S，则m != s，为普通模式，end将到文件末尾。
2、当m存在并且 m == s || m ==S，为分段下载模式。
3、当h缺少, 则h = 0, h1 = 0, 即为h263的处理方式。（h=0 为263格式，否则为h264格式）
4、当h ==0 或者 h1 缺少, 则 h =0, h1 = 0，即为h263的处理方式。
5、当start缺少, start=0 
6、当end 缺少, end = start + max\_transfer\_size(可配，配置项ws\_flv\_flv\_maximum\_drag\_size); 
7、返回403的情况：
  a) start为非数字,或start < 0. 
  b) end为非数字,或end < start 
  c) h为非数字,或h < 0
  d) h1为非数字,或 h1 < h
  e) start或h1 >文件size 
8、修改参数，当end >文件size, end = 文件size。
9、分段下载和普通下载限速不一样，限速方式通过rate\_per\_second配acl进行限速。
10、文件组成分析 
  a) 当请求确定为h263时： 
  head：13固定头。 
  Body： 
    Start 到文件末（普通下载模式） 
    Start到end（分段下载模式）
  b) 当请求确定为h264时： 
  head： 0 到 h1 
  body：
   start 到文件末（普通下载模式）
   start到end（分段下载模式）
  
流程对比:
H263流程对比,
旧版本：head = 固定头部，body = start 到 end。 
新版本：head = 固定头部，body = start 到 end (分段下载模式，不能大于最大分段)

     Body = start 到文件末(普通下载模式)。 

H264流程对比, 
旧版本：head = 固定头部 + 合适的前几个视频音频帧， body = start 到 end。
新版本：head = 0 到 h1，Body = start 到 end（分段下载模式，不能大于最大分段），

         Body = start 到文件末（普通下载模式）。 

### 相关配置:

191.  ws\_flv 
192.  ws\_flv\_h264 
193.  ws\_mp48 
194.  ws\_flv\_all 
195.  ws\_flv\_mp4\_start_name 
196.  ws\_flv\_mp4\_end\_name 
197.  ws\_flv\_mp4\_start\_name\_hash 
198.  ws\_flv\_mp4\_end\_name\_hash 
199.  ws\_flv\_mp4\_start\_time 
200.  ws\_flv\_flv\_start\_time 
201.  ws\_flv\_mp4\_start\_time\_access 
202.  ws\_flv\_flv\_start\_time\_access 
203.  ws\_flv\_mp4\_miss\_before\_store\_complete 
204.  ws\_mp4\_add\_start\_with\_0 
205.  ws\_flv\_mp4\_enable\_range 
206.  ws\_h264\_mp4\_meta 
207.  ws\_pptv\_mp4 
208.  ws\_mp4\_check\_stco 
209.  ws\_nginx\_mp4 
210.  ws\_flv\_add\_meta 
211.  ws\_flv\_mp4\_time\_units 
212.  ws\_flv\_forward\_query 
213.  ws\_flv\_flv\_drag\_for_56 
214.  ws\_flv\_flv\_maximum\_drag\_size 
215.  ws\_flv\_flv56\_download\_mode\_name\_hash 
216.  ws\_flv\_flv56\_module\_press\_mode\_hash 
217.  ws\_flv\_flv56\_module\_head\_offset\_hash 
218.  ws\_flv\_flv56\_download\_mode\_name 
219.  ws\_flv\_flv56\_module\_press\_mode 
220.  ws\_flv\_flv56\_module\_head\_offset 
221.  range\_from\_url 

### 配置说明：

#### 191. ws\_flv

配置描述：
配置flv的acl，匹配上的url允许拖拉。 
配置格式：

	ws_flv allow|deny aclname…

可分频道配置。 
默认配置：
无 

#### 192.     ws\_flv\_h264

配置描述：
新浪h4v文件和普通的flv文件不同，需要文件头的几个特殊的块才能播放。 
此项是配置h4v文件的acl，匹配上的url允许拖拉。 
配置格式：

    ws_flv_h264 allow|deny aclname 

可分频道配置。 
默认配置：
	
	无

#### 193.     ws\_mp4
配置描述：
配置mp4的acl，匹配上的url允许拖拉。 
配置格式：

	ws_mp4 allow … 

可分频道配置。 
默认配置：
无 

#### 194.     ws\_flv\_all

配置描述：
此配置项允许，相当于ws\_flv，ws\_flv\_h264，ws\_mp4三个配置均允许的情况。 
ws\_flv\_all与ws\_flv，ws\_flv\_h264，ws\_mp4三个配置项的关系为：并列关系。 
配置格式：

    ws_flv_all deny|allow aclname 

不可分频道配置。 
默认配置：

	none 

配置例子：
例1：
原来配置.  

	ws_flv allow all 
	ws_flv_h264 allow all 
	ws_mp4 allow all 

现在只需要配置:
	
	ws_flv_all allow all 
  
例2：
配置 

	ws_flv_all allow mp4_2 
	ws_flv allow flv 
	ws_flv_h264 allow h264 
	ws_mp4 allow mp4 

则对mp4_2，flv，h264，mp4均进行视频拖拉。 

#### 195.     ws\_flv\_mp4\_start\_name

配置描述：
配置url中，问号后面start的名字 
配置格式：

	ws_flv_mp4_start_name … 

可分频道配置。 
默认配置：

	start 

#### 196.     ws\_flv\_mp4\_end\_name

配置描述：
配置url中，问号后面end的名字 
配置格式：

	ws_flv_mp4_end_name … 

可分频道配置。 
默认配置：

	end 

配置例子：

	acl ws_flv_test url_regex -i ^http://.*\.flv(|\?.*)$ 
	acl ws_mp4_test url_regex -i ^http://.*\.mp4(|\?.*)$ 
	ws_flv allow ws_flv_test 
	ws_flv deny all 
	ws_mp4 allow ws_mp4_test 
	ws_mp4 deny all 
	ws_flv_mp4_start_name tflvbegin 
	ws_flv_mp4_end_name tflvend 
	ws_flv_mp4_start_time 

#### 197.     ws\_flv\_mp4\_start\_name\_hash

配置描述：
配置url中，问号后面start的名字 
配置格式：

    ws_flv_mp4_start_name_hash "domain" 

可分频道配置。 
默认配置：

	ALL start 

所有没有配置的域名都使用这个值 

#### 198.     ws\_flv\_mp4\_end\_name\_hash

配置描述：
配置url中，问号后面end的名字 
配置格式：

    ws_flv_mp4_end_name_hash "domain" … 

可分频道配置。 
默认配置：

	ALL start 

所有没有配置的域名都使用这个值 
配置例子：

	ws_flv_mp4_start_name_hash domain … 
	ws_flv_mp4_end_name_hash domain … 
	HOST domain 
	ws_flv_mp4_miss_before_store_complete on 
	ws_flv_mp4_start_time on 
	HOST END 

说明： 
ws\_flv和ws\_mp4分别配置flv和mp4的acl，匹配上的url允许拖拉 
ws\_flv\_mp4\_start\_name\_hash和ws\_flv\_mp4\_end\_name\_hash分别是url中，问号后面start和end的名字，可以按域名配置，如果域名(domain)配置为ALL，就是默认值，所有没有配置的域名都使用这个值。默认是 

	ws_flv_mp4_start_name_hash ALL start 
	ws_flv_mp4_end_name_hash ALL end 

如果ws\_flv\_mp4\_miss\_before\_store\_complete是on，则当本地miss时使用原始的url回源，而不是取全部文件。（源必须支持拖拉） 

#### 199.     ws\_flv\_mp4\_start\_time

配置描述：
支持按时间拖拉 
配置格式：
	
	ws_flv_mp4_start_time on/off 

可分频道配置。 
默认配置：

	off 

备注： 
支持按域名配置 
配置例子：

	HOST www.test.com 
	ws_flv_mp4_start_time on 
	HOST END 

如果配置为on，则start的值表示时间(秒)而不是偏移量(字节)。end不再有效，返回的文件从start的时间开始到结束。 

#### 200.     ws\_flv\_flv\_start\_time

配置描述：
告诉squid参数是否是时间拖拉(偏移)，支持全局和按频道配置。  
配置格式：

	ws_flv_flv_start_time on/off 

可分频道配置。 
默认配置：
off 
配置例子：

      ws_flv allow all 
      HOST www.testa.com 
      ws_flv_flv_start_time on 
      HOST END 

#### 201.     ws\_flv\_mp4\_start\_time\_access

配置描述:
当ws\_flv\_mp4\_start\_time配置为on，或者ws\_flv\_mp4\_start\_time\_access配置为allow，则按时间拖拉 
配置格式:

	ws_flv_mp4_start_time_access allow|deny aclname… 

默认配置:

	deny all 

#### 202.     ws\_flv\_flv\_start\_time\_access

配置描述:
当ws\_flv\_flv\_start\_time配置为on，或者ws\_flv\_flv\_start\_time\_access配置为allow，则按时间拖拉 
配置格式:

	ws_flv_flv_start_time_access allow|deny aclname 

默认配置:

	deny all 

#### 203.     ws\_flv\_mp4\_miss\_before\_store\_complete

配置描述：
背景： 
flv和mp4的拖拉功能，会回源请求完整的文件，然后根据用户请求的start和end，返回不同的数据。当start比较大时，如果回源请求全部文件，需要等到文件下载到start位置时，才会返回数据给用户。这种情况下，用户需要等待很长的时间。 
为解决以上用户需要等待很长的时间的问题，当start不是0，并且缓存中没有这个文件(或这个文件正在下载，但还未下载到start的部分,如果配置了ws\_flv\_mp4\_start\_time则下载完之前全部miss)时，发送原始的请求回源(带start和end)，而不是取全部文件。返回的数据不会缓存。 
如果配置成off，当请求的start！=0时，则回源的请求没有start和end，而是取完整的文件，squid将处理后的数据返回给用户，同时缓存整个文件。 
如果配置成on，当请求的start!=0时，则回源的请求有start和end，Squid将源返回的内容直接给客户，不缓存文件。这种情况下需要源支持拖拉功能。 
配置格式：

	ws_flv_mp4_miss_before_store_complete on|off 

可分频道配置。 
默认配置：

	off 

备注： 
1、  源必须支持拖拉功能，否则无法正确拖拉。 
配置例子：

	HOST www.test.com 
	ws_flv_mp4_miss_before_store_complete on 
	HOST END 
	ws_flv_mp4_miss_before_store_complete off 

www.test.com发送原始的请求回源(带start和end)。 

#### 204.     ws\_mp4\_add\_start\_with\_0

配置描述:
该配置未找到相关文档，配置描述待补充:  
配置格式:

	ws_mp4_add_start_with_0 on|off 

可分频道配置 
默认配置:

	off 

#### 205.     ws\_flv\_mp4\_enable\_range

配置描述:
   该配置未找到相关文档，配置描述待补充 
配置格式:

	ws_flv_mp4_enable_range on|off 

可分频道配置 
默认配置:

	off 

#### 206.     ws\_h264\_mp4\_meta

配置描述：
匹配上acl的请求，返回视频的metadata信息 
配置格式：

	ws_h264_mp4_meta allow ... 

可分频道配置。 
默认配置：

	无 

#### 207.     ws\_pptv\_mp4

配置描述:
配置为on，表示使用pptv的pm4拖拉算法进行拖拉 
拖拉的其他相关信息，如start/end名称，时间拖拉/字节拖拉等，配置不变 
配置格式:

	ws_pptv_mp4 on|off 

可分频道配置 
默认配置:

	off 

#### 208.     ws\_mp4\_check\_stco

配置描述：
背景： 
squid对mp4头的处理方式： 
1.将moov头保存到一个连续的内存块中 
2.解析moov头，以指针的方式保存解析得到的各个atom的位置 
3.计算拖拉处理后的数据 
4.根据拖拉的结果，重写moov头，将新的数据按atom写回原来的位置，得到的moov与原来的大小一致且每个atom的大小和位置也一致 
有些mp4文件在拖拉后，生成新的atom比原来的atom大(stsc块),写回原来的atom位置时，需要丢弃后面一部分的数据，导致视频文件出错。 
配置了该项，则在mp4拖拉之前，先将mp4头扩展，避免出现头改写后数据丢失 
配置格式：

	ws_mp4_check_stco on|off 

可分频道配置。 
默认配置：

	off 

#### 209.     ws\_nginx\_mp4

配置描述:
当配置为on，且配置了mp4时间拖拉时，采用nginx的拖拉算法进行拖拉配置为off时，兼容旧版本 
配置格式:

	ws_nginx_mp4 on|off 

默认配置:

	off 

#### 210.     ws\_flv\_add\_meta

配置描述:
当配置为on时，对flv和h264的拖拉，除了返回视频数据外，还返回重构的meta信息。 
配置格式:

	ws_flv_add_meta on|off 

可分频道配置 
默认配置:

	off 

#### 211.     ws\_flv\_mp4\_time\_units

配置描述:
   当配置flv或mp4时间拖拉时，从url中取出的start（或end)，将乘以ws\_flv\_mp4\_time\_units1  
配置格式:

	ws_flv_mp4_time_units float1 

可分频道配置 
默认配置:

	1 

#### 212.     ws\_flv\_forward\_query

配置描述：
背景： 
目前视频拖拉回源会去掉问号后面所有的内容，个别客户去掉问号后面所有的内容之后由于缺少key无法访问。 
配置视频拖拉回源是否加查询串，配置为on则回源只去掉拖拉相关的查询串，保留其他的参数。 
配置格式：

	ws_flv_forward_query on|off 

可分频道配置。 
默认配置：

	off 

备注： 
1、  默认配置为off，即视频拖拉回源会去掉问号后面的所有内容。 
2、  该配置只修改视频拖拉回源的url，缓存的url仍然是不带查询串的。 
3、  去问号配置项url\_cut\_question\_mark在这种情况下对视频拖拉回源的url不起作用，也就是说配置了flv\_forward\_query为on，如果去掉了url中拖拉相关的关键字之后问号后面还有带其他参数，那么即使配置了url\_cut\_question\_mark为on，回源也同样带问号及非拖拉的其他参数。 

#### 213.     ws\_flv\_flv\_drag\_for\_56

配置描述：
是否对56用户的拖拉行为执行特殊处理 
配置格式：

	ws_flv_flv_drag_for_56   on/off 

可分频道配置。 
默认配置：

	off  

备注： 
1、  为on时，可以对？后面参数配置限速 
2、  支持全局和按频道配置，频道配置级别高于全局配置 

#### 214.     ws\_flv\_flv\_maximum\_drag\_size

配置描述：
当客户端请求链接中，带问号参数中无end参数时，

	end = start + ws_flv_flv_maximum_drag_size 

配置格式：

	ws_flv_flv_maximum_drag_size 

可分频道配置。 
默认配置：

	10000 KB 

配置例子：

	acl 56ratemode hash_url_regex 192.168.21.227  -i ^http://192.168.21.227:81/.*m=s.* 
	rate_per_second  100 KB  allow 56ratemode 
	ws_flv_flv_maximum_drag_size 10000 KB 
	HOST www.56.com 
	ws_flv_flv_drag_for_56 on 
	HOST END 

#### 215.     ws\_flv\_flv56\_download\_mode\_name\_hash

配置描述:
   56视频拖拉客户的需求配置，具体说明作用，暂时没找到相关文档 
配置格式:

	ws_flv_flv56_download_mode_name_hash string 

默认配置:

	none 

#### 216.     ws\_flv\_flv56\_module\_press\_mode\_hash

配置描述:
56视频拖拉客户的需求配置，具体说明作用，暂时没找到相关文档 
配置格式:

	ws_flv_flv56_module_press_mode_hash string 

默认配置:

	none 

#### 217.     ws\_flv\_flv56\_module\_head\_offset\_hash

配置描述:
56视频拖拉客户的需求配置，具体说明作用，暂时没找到相关文档 
配置格式:

	ws_flv_flv56_module_head_offset_hash string7 

默认配置:

	none 

#### 218.     ws\_flv\_flv56\_download\_mode\_name

配置描述：
56用户视频拖拉请求中的下载模式字段名称。 
配置格式：

	ws_flv_flv56_download_mode_name m2 S1 t 

可分频道配置。 
默认配置：

	m 

配置例子：

	http://www.56.com/1.flv?start=1&m=s 

#### 219.     ws\_flv\_flv56\_module\_press\_mode

配置描述：
56用户视频拖拉请求中的压缩模式字段名称。 
配置格式：

	ws_flv_flv56_module_press_mode h 

可分频道配置。 
默认配置：

	h 

配置例子：

	http://www.56.com/1.flv?start=1&h=09  

#### 220.     ws\_flv\_flv56\_module\_head\_offset

配置描述：
56用户视频拖拉请求中的头部偏移量字段名称。 
配置格式：

	ws_flv_flv56_module_head_offset h1 

可分频道配置。 
默认配置：

	h1 

配置例子：

	http://www.56.com/1.flv?start=1&h1=0 

#### 221.     range\_from\_url

配置描述：
请求没有Range头而且range\_from\_url允许的情况下：  
1）无论0range标识是否存在，则从最后一级文件名xxx.mp4的前面取出rangestart和rangeend 
2）rangestart或者rangeend为非法数字，则按请求url处理，不添加range头 
3）rangestart<=0，则A=0，否则A=rangestart 
4）rangeend < A，则返回400 
5）如果rangeend =0 ，则构造Range: bytes=A-，可以从A取到文件结尾 
6）否则，构造Range: bytes=A-rangeend 
7）从请求url中去掉rangestart和rangeend，如果存在0range，也要去掉 
8）rangestart > content-length，返回416 
9）rangeend > content-length，则返回从rangestart到文件结尾的内容.  
注： 
1）squid对range请求的相关配置，对此依然有效 
2）此处理在内部重定向之前，对升级前的内部重定向配置无影响 
3）经过处理后，缓存，回源，日志，流量等，记录的都是http://ip:port/segno//xxx.mp4?key=xxx这样的url 
4) 改进方向：可以动态配置rangestart和rangeend在请求url的位置 
配置格式：

	range_from_url allow/deny aclname，默认None，可以分频道配置 

可分频道配置。 
默认配置：

	None 

配置例子:
1）请求http://aaa.aa.com/1.html，则按原来url处理，不处理range   
2）请求http://aaa.aa.com/aaaa/300/11111.mp40range，则按原来url处理，不处理range 
3）请求http://aaa.aa.com/20/300/11111.mp40range，带Range: bytes=1000-2000，则按原来url处理，返回1000-2000的数据 
4）请求http://aaa.aa.com/-1/300/11111.mp40range，则url变成http://aaa.aa.com/11111.mp4，返回0-300的数据 
5）请求http://aaa.aa.com/1500/300/11111.mp40range，返回400响应码 
6）请求http://aaa.aa.com/1500/0/11111.mp40range，则url变成http://aaa.aa.com/11111.mp4，返回从1500到文件结尾的数据 
7）假设文件长度为4096，请求http://aaa.aa.com/5500/9300/11111.mp40range，则url变成http://aaa.aa.com/11111.mp4，返回416 
8）假设文件长度为4096，请求请求http://aaa.aa.com/2500/9300/11111.mp40range，则url变成http://aaa.aa.com/11111.mp4，返回2500-4095的数据  
