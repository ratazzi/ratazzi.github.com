---
layout: post
title: "基于 Linux 的 Time Machine 备份 － 针对 Lion"
category: 
tags: ['Time Machine', 'Lion', ' AFP', 'netatalk']
---

网上类似的文章不少，我就不重复了，捡重要的写，这里提供几篇靠谱的文章
	http://www.gracecode.com/archives/3057/
	http://plus-alpha-space.cocolog-nifty.com/blog/2011/07/how-to-install.html
	http://bit.ly/pWeMZq
	
Snow Leopard 及以前的系统貌似可以直接使用 samba 之类的搞定，我没试过，Lion 必须使用 AFP 3.3 的才可以，但是现在主流的系统都是 netatalk 2.1.x 不支持，上面的第二个链接是讲怎么在 ubuntu 下装 netatalk 2.2 的，第三个是针对 freenas 的。
 我在 ubuntu 10.10 下测试通过，freenas 不大了解没有试，还有一点就是 `/etc/netatalk/AppleVolumes.default` 文件中的
	cnidscheme:cdb # cdb 需要改成 dbd，否则每次连接都会有警告

另外就是第一个链接中安装的几个包在 ubuntu 10.10 中没有，我装的是
	sudo apt-get install netatalk avahi-daemon

`nss-mdns` 我没找到这个包，但是发现系统本来就有 `nsswitch.conf `这个文件，直接编辑就可以了，上截图

<img width="640" src="http://cl.ly/0i36420Z3e2Q2h3m0u3M/Screen%20Shot%202011-08-14%20at%205.56.54%20PM.png" alt="Time Machine">
