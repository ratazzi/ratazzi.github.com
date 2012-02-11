---
layout: post
title: "关于 PAC 文件远程 DNS"
category: 
tags: ['pac', 'firefox', 'socks']
---
{% include JB/setup %}

PAC 文件配合 firefox, ssh 简直是神器，但是在启用远程 DNS 的时候还是会有一些网站不能访问，为了畅通无阻，还需要改一下

	/* 在 PAC 文件中必须以 SOCKS5 返回才能让 ff 使用远程 DNS 解析 */
	if (shExpMatch(url, ".flickr.com/")) { return "SOCKS5 127.0.0.1:7777"; }
	
	/* 这个是针对没有 www 之类的网站 */
	if (dnsDomainIs(host, "twitter.com")) { return "SOCKS5 127.0.0.1:7777"; }