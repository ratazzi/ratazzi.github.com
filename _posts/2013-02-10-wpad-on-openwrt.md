---
layout: post
title: "WPAD 在 OpenWRT 上的配置（PAC 文件自动分发）"
category:
tags: ['OpenWrt', 'WPAD', 'PAC']
---
{% include JB/setup %}

自从 OpenVPN 沦陷以后，就不得不回到用代理的时代，之前刷好 OpenWrt 的路由器也闲置了，因为不能无痛翻墙，代理程序跑在路由器上我觉得已经没有什么意义了，因为每个设备都需要去配置代理，直到我知道了 [WPAD][wpad] 这个协议。

第一次知道这个协议是在 [@Paveo][paveo] 的博客上看到的，这个协议实现 pac 文件的自动分发，网络上关于这个协议的内容不是很多，我也只知道可以通过 Dnsmasq 的 DHCP 来实现。废话不多说，`/etc/config/dhcp`
    
    # ...
    list dhcp_option '252,http://example.com/proxy.pac'

`252` 不知道什么意思，猜测是某个类型的代码之类的，剩下的就是在设备上配置，好吧我承认还是要配置，但是比起之前还是好了不少，因为不用输入 pac 文件的地址了，在 Mac 上，勾选 `Auto Proxy Discovery` 即可，iOS 上设置代理为自动，URL 留空即可。

当然这比起之前 VPN 结合 chnroutes 还是逊色了不少，但是至少未越狱的 iOS 设备上用起来就爽很多了。

[wpad]: http://zh.wikipedia.org/wiki/代理自动配置 "WPAD"
[paveo]: https://w3.owind.com/pub/wpad-and-falcop/
