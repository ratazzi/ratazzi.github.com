---
layout: post
title: "优化 VPN 和 chnroutes"
category: 
tags: ['OpenVPN', 'chnroutes', 'iptables', 'squid', 'Dnsmasq']
---
{% include JB/setup %}

对于天朝这悲剧网络来说，我是被迫这样的，原因就不说了，主要优化以下几点：

* VPN 主要是服务器端，并且有局限性
* 域名解析，会用到 [Dnsmasq][dnsmasq]
* [chnroutes][chnroutes]，主要是路由表

### VPN 服务器端的优化
可以加个 [squid][squid] 代理，将 VPN 的 http 流量交给 [squid][squid] 来处理，这要用户比较多才有效果，使用如下规则将 http 流量转给 [squid][squid]：

	iptables -t nat -A PREROUTING -s 10.8.0.0/24 -i tun0 -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 3128
其中 10.8.0.0 是 VPN 所在的网段，3128 是 [squid][squid] 的端口。

### 域名解析的优化
因为 DNS 污染的缘故，我不得不默认使用 OpenDNS 或者 Google DNS 之类的，这样在解析国内域名的时候就会解析到很慢的 ip 上，造成上国内某些网站相当慢，比如视频网站、迅雷之类的。很多人都会用 [Dnsmasq][dnsmasq] 来搭建一个本地的 DNS 服务器来定制常用的域名解析，这神器相当轻量，可以指定某些域名用指定 DNS 解析，或者直接作为增强版的 hosts 文件。

我的方案是使用本地、香港、台湾以及 OpenDNS 的 DNS 服务器去解析同一个域名，然后作简单的 ping 测试，然后选择最快的服务器，当然这个过程是用脚本来完成的，如何获得常用的需要优化的域名，一个很简单的办法是先使用 [Dnsmasq][dnsmasq] 一段时间，一定要记得打开日志，然后用一段时间后从日至中提取域名。

我自己写了一个 [dnsmasq.gen][dnsmasq_gen] 的工具，这个工具能够辅助你，部分还是得手工，毕竟 ping 测试的响应速度不代表真实网速，比如我这里相同响应时间，台湾的服务器能够跑满带宽而香港的则不行。

### 最后是路由表的优化
仅仅使用 [chnroutes][chnroutes] 生成的路由表是不够的，因为该路由表仅仅包含了大陆的 ip，而香港、台湾不在里面，如果你的 VPN 服务器是美国的话，那么就必须得优化了，一般来说直连香港、台湾是比较快的，某些日本服务器也是如此，当然也不是一股脑的将香港和台湾的 ip 也加进来而是有选择性的，这里是针对前面域名解析优化以后，将 `dnsmasq.conf` 中的 ip 提取出来然后取得 ip 信息后再对比使用 VPN 快还是直连快，最后确定是否加入到路由表中。

[dnsmasq_gen]: http://github.com/ratazzi/dnsmasq.gen "dnsmasq.gen"
[chnroutes]: http://code.google.com/p/chnroutes/ "chnroutes"
[dnsmasq]: http://www.thekelleys.org.uk/dnsmasq/doc.html "Dnsmasq"
[squid]: http://www.squid-cache.org/ "squid"
