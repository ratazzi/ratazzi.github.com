---
layout: post
title: "OpenWrt 捣鼓记"
category: 
tags: ['OpenWrt', 'OpenVPN']
---
{% include JB/setup %}

事情的起因是我不爽 OpenWrt 默认的 shell，因为一些洁癖，平常我都用 zsh，最起码也得用 bash，然后在 n 次由于后还是忍不住的改了 `/etc/passwd` 的默认 shell，当时完全没有想到的是这玩意儿影响那么大。后果是重启之后 ssh 验证不能了，本来是用 key 验证的，现在却让我输密码，输就输吧， tmd 还不认。

然后就是一系列的悲剧，reset 完全不起作用，无奈去 [V2EX][v2ex] 求助，因为刷的官方固件，木有 web 界面，telnet 也被系统停掉了。在 [V2EX](http://www.v2ex.com/t/28627) 几位同学提供的方案之后都未果，甚至用系统自带的 ash 去替换了优盘上的 bash，也就是替换了修改之后 `/etc/passwd` 里的默认 shell 可执行文件，依然未果。甚至想到了用串口数据线直接重新刷机来搞的，不过我实在没那天份，在研究了半天之后发现对于我来说并不是那么容易上手的，反正目前路由器也是刷好的，OpenVPN 也是正常使用的，就想着慢慢研究，后面来个有挑战性的串口刷机修砖，现在也不影响使用，OpenVPN 的配置都是在优盘上的，随时可以改，只是失去控制权让我不爽。

今天突然想到可以利用 OpenVPN 的 up，down 脚本执行外部命令，而且是 root 权限，于是又开始折腾，反复插拔优盘之后终于用 sed 还原了 `/etc/passwd` 文件，重启路由器，ssh 连上去，成功了，重新拿到 root 权限，那个激动啊，而且有点当了一回 cracker 的感觉。

最后非常感谢 [V2EX][v2ex] 几位同学提供各种解决方案。

[v2ex]: http://www.v2ex.com/ "V2EX"