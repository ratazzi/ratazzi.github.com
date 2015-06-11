---
layout: post
title: "Mac OS X 的全局代理"
category: 
tags: ['Mac OS X', 'pac', 'socks']
---

Mac 上可以使用 pac 文件自动设置全局代理，相当的方便，但是有个小问题，设置为 SOCKS5 的时候，Firefox，Chrome 不会使用远程 DNS， 而 Safari 不可以使用 SOCKS5 这种方式，于是就有了下面的解决方案：

{% highlight javascript %}
function FindProxyForURL(url, host) 
{
    var proxy = "SOCKS5 127.0.0.1:7777;SOCKS 127.0.0.1:7777";
    if (shExpMatch(url, "*.37signal.com/*")) { return proxy; }
    if (dnsDomainIs(host, "twitter.com")) { return proxy; }
    if (dnsDomainIs(host, "github.com")) { return proxy; }
    if (dnsDomainIs(host, "img.ly")) { return proxy; }
}
{% endhighlight %}
	
这样三个浏览器都可以正常使用了。

补充：也可以写个 php 之类的脚本根据 User-Agent 动态的判断是 SOCKS5 还是 SOCKS，每个浏览器在启动时会带着 UA 去请求。
