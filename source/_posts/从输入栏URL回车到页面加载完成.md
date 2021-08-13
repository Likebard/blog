---
title: 从输入栏URL回车到页面加载完成
date: 2021-02-13 10:31:40
tags: web
---

1. 打开一个Tab页，新开一个进程(某些情况下多个tab会合并进程)。每一个tab页可以看作是浏览器内核进程，这个进程是多线程的，几大类子线程:
    - GUI线程
    - JS引擎线程
    - 事件触发线程
    - 定时器线程
    - 网络请求线程
其中JS引擎就能说明我们为什么常说JS是单线程的。
<!--more-->
2. 首先进行URL解析，URL中出现了中文会转为Unicode字符（encodeURI()是Javascript中真正用来对URL编码的函数。）。按照本地host文件 > 询问dns服务器，解析对应的IP地址。
   - URL的组成部分：
   - protocol，协议头，譬如有http，ftp等

   - host，主机域名或IP地址

   - port，端口号

   - path，目录路径

   - query，即查询参数

   - fragment，即 #后的hash值，一般用来定位到某个位置， a标签的href中"#abc"，点击后会定位到id为abc的元素上，或者name为abc的a标签上
  
3. 查到ip后跟服务器建立连接，三次握手四次挥手
   - 主机发送SYN = 1的TCP包给服务器


4. 建立tcp/ip连接后就返回html文件，判断是否在缓存里，有就返回304,让浏览器直接读取缓存，没有的话就去后台拿
    - 强缓存
    - 协商缓存
    - from disk cache和from memory cache
    - 启发式缓存： 如果Expires, Cache-Control: max-age, 或 Cache-Control:s-maxage 都没有在响应头中出现, 并且也没有其它缓存的设置, 那么浏览器默认会采用一个启发式的算法, 通常会取响应头的Date_value - Last-Modified_value值的10%作为缓存时间.
5. tcp/ip的并发限制，浏览器对同一域名下的并发连接是有限制的。http1中一个资源下载就对应一个tcp/ip请求，http2中还有多路复用机制，需要归纳比较http1.0-http3的特点,还有https


6. 解析页面流程
   - ![avatar](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0cd9OTY4K2QLZVZpkPy4vzKlicFpfcdxPEhnYe3dCFpu1auzwXKtXQaVvu29OrhDibpmIK6d36yMa2g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)