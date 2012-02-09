---
layout: post
title: "Install OpenVPN and OpenWrt on WR703N"
category: 
tags: []
---
{% include JB/setup %}

终究还是买了个，同时买了个 U 盘，开始刷的官方固件，我不知道不带 web 界面，重新刷了个带 web 界面的，折腾了下
结果因为空间不足以安装挂载 U 盘的软件包，无奈刷回了官方固件，大概也知道了怎么配置网络，两个固件第一个可以直接在
web 界面刷，第二个只能 telnet 到路由器刷
`mtd -r write openwrt-xxx.bin firmware`

重启后 telnet 配置网络，首先要可以上网先，我没有配置 PPPOE，而是直接接的之前的路由器
/etc/config/network:

    config 'interface' 'lan'
        option 'ifname' 'eth0'
        option 'proto' 'dhcp'
    config 'interface' 'wifi'
        option 'proto' 'static'
        option 'ipaddr' '192.168.2.1'
        option 'netmask' '255.255.255.0'

/etc/config/wireless

    config 'wifi-iface'
        option 'device' 'radio0'
        option 'network' 'wifi'
        option 'mode' 'ap'
        option 'ssid' 'OpenWrt'
        option 'encryption' 'psk2'
        option 'secret' 'password'
        option 'key' 'key'

/etc/config/dhcp

    config dhcp wifi
        option 'interface' 'wifi'
        option 'start' '100'
        option 'limit' '150'
        option 'leasetime' '12h'

/etc/rc.local

    iptables -t nat -F
    iptables -t filter -F
    iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -A FORWARD -s 192.168.2.0/24 -j ACCEPT
    iptables -A FORWARD -j REJECT
    iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -o eth0 -j MASQUERADE
    iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -o tun0 -j MASQUERADE

重启相关服务：

    /etc/init.d/network restart
    ifup wifi
    wifi
    /etc/init.d/firewall disable
    /etc/init.d/firewall stop
    /etc/init.d/dnsmasq restart

格式化 U 盘（在别的 linux 机器上进行，官方建议关闭 journal 以提高性能）：

    mkfs.ext4 -O ^has_journal,extent /dev/sda1

安装挂载 U 盘需要的软件包：

    opkg install kmod-usb-storage block-mount kmod-usb-storage-extras kmod-usb2 \ 
    kmod-nls-base kmod-nls-iso8859-1 kmod-nls-utf8 kmod-fs-ext4

配置 U 盘自动挂载，自己编辑 /etc/config/fstab 一看就知道了，注意 enabled，另外在 
`/etc/opkg.conf` 中增加 `dest usb /mnt/usb`，安装软件包：

    opkg install kmod-tun libopenssl zlib ldconfig
    opkg --dest usb install liblzo openvpn
    opkg --dest usb install vim-full

配置 OpenVPN：

    config openvpn linode
        option client 1
        option script_security 2
        option max_routes 2500
        option dev tun
        option proto udp
        option remote 173.230.148.194 1194 
        option resolv_retry infinite
        option ca /mnt/usb/etc/openvpn/ca.crt
        option cert  /mnt/usb/etc/openvpn/wr703n.crt
        option key  /mnt/usb/etc/openvpn/wr703n.key
        option comp_lzo 1
        option persist_key 1
        option persist_tun 1
        option nobind 1
        option float 1
        option remote_cert_tls server
        option shaper 5242880
        option ping 10
        option ping_restart 60
        option verb 6
        option status /mnt/usb/var/log/openvpn/openvpn-status.log
        option log /mnt/usb/var/log/openvpn/openvpn.log
        list route '101.0.0.0 255.255.252.0 net_gateway 5'

需要注意的是，`/etc/init.d/openvpn` 中没有 max_routes 这个选项，需要自己加，尽量
加在前面否则生成的配置文件可能会不正确，路由部分的引号是必须的，我就因为这浪费了
不少时间。

参考：  
http://lgallardo.com/en/2011/09/08/configurar-openvpn-en-openwrt/
