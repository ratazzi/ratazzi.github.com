---
layout: post
title: "关于最近 OpenVPN 被干扰的分析"
category:
tags: ['OpenVPN']
---
{% include JB/setup %}

## OpenVPN 连接的建立

* 首先客户端发送一个 opcode 为 `P_CONTROL_HARD_RESET_CLIENT_V2` 的包，这个数据包有一个 sessionid
* 而服务器应该回应一个 opcode 为 `P_CONTROL_HARD_RESET_SERVER_V2` 的包，并且包含客户端的 sessionid，同时也会有一个服务端的 sessionid
* 然后客户端发送 opcode 为 `P_ACK_V1` 的 ACK，还有一个 opcode 为 `P_CONTROL_V1` 的 control 类型包

后面一直是 opcode 为 `P_CONTROL_V1`, `P_ACK_V1`, `P_DATA_V1` 的数据包交互，连接建立完成之后基本上都是 opcode 为 `P_DATA_V1` 的数据包了，因为关键在前面几个包，后面也比较复杂就没研究下去，`P_CONTROL_HARD_RESET_CLIENT_V2` 和 `P_CONTROL_HARD_RESET_SERVER_V2` 这两个类型的包只有一次。

更详细的连接协议参考：

<http://blog.csdn.net/dog250/article/details/6647457>

## 数据包分析

OpenVPN 协议的 opcode 部分注解：

    P_CONTROL_HARD_RESET_CLIENT_V2   16 进制为 0x38 (其实是 0x38 >> 3 = 7)
    P_CONTROL_HARD_RESET_SERVER_V2   16 进制为 0x40 (0x40 >> 3 = 8)
    P_ACK_V1                         16 进制为 0x28 (0x28 >> 3 = 5)
    P_CONTROL_V1                     16 进制为 0x20 (0x20 >> 3 = 4)
    P_DATA_V1                        16 进制为 0x30 (0x30 >> 3 = 6)
    UDP 数据中第一字节高五位为 opcode, 低三位为 kid，算法参考下面链接
    https://gist.github.com/4239543

接下来看一份正常情况下的数据包：

    # 192.14.200.95 为客户端
    # 222.222.222.222 为服务端，OpenVPN 端口为 6671

    21:25:49.199836 IP (tos 0x0, ttl 51, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.14.200.95.45323 > 222.222.222.222.6671: UDP, length 14
        0x0000:  4500 002a 0000 4000 3311 85ac c00e c85f  E..*..@.3......_
        0x0010:  dede dede b10b 1a0f 0016 0d1d 3836 a1fa  ............86..
        0x0020:  eaca 9577 0b00 0000 0000                 ...w......
    # 第一行的最后四字节为源 ip 地址
    # 第二行的前四字节为 ip 目标地址，中间八字节为 UDP 头部
    # 第二行的最后四字节的第一字节即为 opcode 和 kid
    # 紧接着 0x38 后面的 36 a1 fa ea ca 95 77 是客户端的 sessionid

    21:25:49.235381 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 54)
        222.222.222.222.6671 > 192.14.200.95.45323: UDP, length 26
        0x0000:  4500 0036 0000 4000 4011 78a0 dede dede  E..6..@.@.x.....
        0x0010:  c00e c85f 1a0f b10b 0022 c24a 4034 57c1  ..._.....".J@4W.
        0x0020:  c9c3 41ab 9201 0000 0000 36a1 faea ca95  ..A.......6.....
        0x0030:  770b 0000 0000                           w.....
    # 紧接着 0x40 后面是服务端的 sessionid，最右面可以看到可以看到客户端的 sessionid

    21:25:49.455169 IP (tos 0x0, ttl 51, id 0, offset 0, flags [DF], proto UDP (17), length 50)
        192.14.200.95.45323 > 222.222.222.222.6671: UDP, length 22
        0x0000:  4500 0032 0000 4000 3311 85a4 c00e c85f  E..2..@.3......_
        0x0010:  dede dede b10b 1a0f 001e b816 2836 a1fa  ............(6..
        0x0020:  eaca 9577 0b01 0000 0000 3457 c1c9 c341  ...w......4W...A
        0x0030:  ab92                                     ..
    # 0x28 即 P_ACK_V1

    21:25:49.459242 IP (tos 0x0, ttl 51, id 0, offset 0, flags [DF], proto UDP (17), length 142)
        192.14.200.95.45323 > 222.222.222.222.6671: UDP, length 114
        0x0000:  4500 008e 0000 4000 3311 8548 c00e c85f  E.....@.3..H..._
        0x0010:  dede dede b10b 1a0f 007a 5b93 2036 a1fa  .........z[..6..
        0x0020:  eaca 9577 0b00 0000 0001 1603 0100 7101  ...w..........q.
        0x0030:  0000 6d03 0150 c33f dd67 adcd 87ae adee  ..m..P.?.g......
        0x0040:  efcd 1fe7 5f3f 719b 5492 2b6b 72ef 752a  ...._?q.T.+kr.u*
        0x0050:  4276 d0a0 6e00 003a c022 c021 0039 0038  Bv..n..:.".!.9.8
        0x0060:  0035 c01c c01b 0016 0013 000a c01f c01e  .5..............
        0x0070:  0033 0032 009a 0099 002f 0096 0005 0004  .3.2...../......
        0x0080:  0015 0012 0009 0014 0011 0008 0006       ..............
    21:25:49.459436 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 50)
        222.222.222.222.6671 > 192.14.200.95.45323: UDP, length 22
        0x0000:  4500 0032 0000 4000 4011 78a4 dede dede  E..2..@.@.x.....
        0x0010:  c00e c85f 1a0f b10b 001e c246 2834 57c1  ..._.......F(4W.
        0x0020:  c9c3 41ab 9201 0000 0001 36a1 faea ca95  ..A.......6.....
        0x0030:  770b                                     w.
    21:25:49.461007 IP (tos 0x0, ttl 51, id 0, offset 0, flags [DF], proto UDP (17), length 60)
        192.14.200.95.45323 > 222.222.222.222.6671: UDP, length 32
        0x0000:  4500 003c 0000 4000 3311 859a c00e c85f  E..<..@.3......_
        0x0010:  dede dede b10b 1a0f 0028 e5f2 2036 a1fa  .........(...6..
        0x0020:  eaca 9577 0b00 0000 0002 0003 00ff 0201  ...w............

干扰之后客户端的数据包：

    reading from file client.bin, link-type EN10MB (Ethernet)
    19:28:11.222673 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede a7fd 1a0d 0016 0612 3812 da35  ............8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:28:13.463859 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede a7fd 1a0d 0016 0612 3812 da35  ............8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:28:17.943511 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede a7fd 1a0d 0016 0612 3812 da35  ............8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:28:25.966360 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede a7fd 1a0d 0016 0612 3812 da35  ............8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:28:42.052315 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede a7fd 1a0d 0016 0612 3812 da35  ............8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:29:13.336863 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.60342 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede ebb6 1a0d 0016 0612 386d 992b  ............8m.+
        0x0020:  fbe8 b60b f300 0000 0000                 ..........
    19:29:15.511871 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.60342 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede ebb6 1a0d 0016 0612 386d 992b  ............8m.+
        0x0020:  fbe8 b60b f300 0000 0000                 ..........
    19:29:19.857678 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.60342 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede ebb6 1a0d 0016 0612 386d 992b  ............8m.+
        0x0020:  fbe8 b60b f300 0000 0000                 ..........
    19:29:27.423379 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.60342 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede ebb6 1a0d 0016 0612 386d 992b  ............8m.+
        0x0020:  fbe8 b60b f300 0000 0000                 ..........
    19:29:43.571147 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.60342 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede ebb6 1a0d 0016 0612 386d 992b  ............8m.+
        0x0020:  fbe8 b60b f300 0000 0000                 ..........
    19:30:15.606582 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.46332 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede b4fc 1a0d 0016 0612 387f 7ff2  ............8...
        0x0020:  518f 49a9 7900 0000 0000                 Q.I.y.....
    19:30:18.067324 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.46332 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede b4fc 1a0d 0016 0612 387f 7ff2  ............8...
        0x0020:  518f 49a9 7900 0000 0000                 Q.I.y.....
    19:30:22.985966 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.46332 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede b4fc 1a0d 0016 0612 387f 7ff2  ............8...
        0x0020:  518f 49a9 7900 0000 0000                 Q.I.y.....
    19:30:30.100068 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.46332 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede b4fc 1a0d 0016 0612 387f 7ff2  ............8...
        0x0020:  518f 49a9 7900 0000 0000                 Q.I.y.....
    19:30:46.099408 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        192.168.2.153.46332 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 34d9 c0a8 0299  E..*..@.@.4.....
        0x0010:  dede dede b4fc 1a0d 0016 0612 387f 7ff2  ............8...
        0x0020:  518f 49a9 7900 0000 0000                 Q.I.y.....
    # 可以看到都是客户端发起的 0x38 的请求，没有回应

干扰之后服务器端的数据包：

    19:28:11.392071 IP (tos 0x0, ttl 50, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        183.14.209.53.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 3211 7dd6 b70e d135  E..*..@.2.}....5
        0x0010:  dede dede a7fd 1a0d 0016 5872 3812 da35  ..........Xr8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:28:11.427092 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 54)
        222.222.222.222.6669 > 183.14.209.53.43005: UDP, length 26
        0x0000:  4500 0036 0000 4000 4011 6fca dede dede  E..6..@.@.o.....
        0x0010:  b70e d135 1a0d a7fd 0022 cb20 4086 7656  ...5....."..@.vV
        0x0020:  36b3 2fe8 0501 0000 0000 12da 3516 380d  6./.........5.8.
        0x0030:  d7e4 0000 0000                           ......
    19:28:13.630993 IP (tos 0x0, ttl 50, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        183.14.209.53.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 3211 7dd6 b70e d135  E..*..@.2.}....5
        0x0010:  dede dede a7fd 1a0d 0016 5872 3812 da35  ..........Xr8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:28:13.631324 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 54)
        222.222.222.222.6669 > 183.14.209.53.43005: UDP, length 26
        0x0000:  4500 0036 0000 4000 4011 6fca dede dede  E..6..@.@.o.....
        0x0010:  b70e d135 1a0d a7fd 0022 cb20 4086 7656  ...5....."..@.vV
        0x0020:  36b3 2fe8 0501 0000 0000 12da 3516 380d  6./.........5.8.
        0x0030:  d7e4 0000 0000                           ......
    19:28:17.848064 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        222.222.222.222.6669 > 183.14.209.53.43005: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 6fd6 dede dede  E..*..@.@.o.....
        0x0010:  b70e d135 1a0d a7fd 0016 cb14 4086 7656  ...5........@.vV
        0x0020:  36b3 2fe8 0500 0000 0000                 6./.......
    19:28:18.110017 IP (tos 0x0, ttl 50, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        183.14.209.53.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 3211 7dd6 b70e d135  E..*..@.2.}....5
        0x0010:  dede dede a7fd 1a0d 0016 5872 3812 da35  ..........Xr8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:28:18.110322 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 50)
        222.222.222.222.6669 > 183.14.209.53.43005: UDP, length 22
        0x0000:  4500 0032 0000 4000 4011 6fce dede dede  E..2..@.@.o.....
        0x0010:  b70e d135 1a0d a7fd 001e cb1c 2886 7656  ...5........(.vV
        0x0020:  36b3 2fe8 0501 0000 0000 12da 3516 380d  6./.........5.8.
        0x0030:  d7e4                                     ..
    19:28:25.233404 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        222.222.222.222.6669 > 183.14.209.53.43005: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 6fd6 dede dede  E..*..@.@.o.....
        0x0010:  b70e d135 1a0d a7fd 0016 cb14 4086 7656  ...5........@.vV
        0x0020:  36b3 2fe8 0500 0000 0000                 6./.......
    19:28:26.133021 IP (tos 0x0, ttl 50, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        183.14.209.53.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 3211 7dd6 b70e d135  E..*..@.2.}....5
        0x0010:  dede dede a7fd 1a0d 0016 5872 3812 da35  ..........Xr8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:28:26.133348 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 50)
        222.222.222.222.6669 > 183.14.209.53.43005: UDP, length 22
        0x0000:  4500 0032 0000 4000 4011 6fce dede dede  E..2..@.@.o.....
        0x0010:  b70e d135 1a0d a7fd 001e cb1c 2886 7656  ...5........(.vV
        0x0020:  36b3 2fe8 0501 0000 0000 12da 3516 380d  6./.........5.8.
        0x0030:  d7e4                                     ..
    19:28:41.687249 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        222.222.222.222.6669 > 183.14.209.53.43005: UDP, length 14
        0x0000:  4500 002a 0000 4000 4011 6fd6 dede dede  E..*..@.@.o.....
        0x0010:  b70e d135 1a0d a7fd 0016 cb14 4086 7656  ...5........@.vV
        0x0020:  36b3 2fe8 0500 0000 0000                 6./.......
    19:28:42.218759 IP (tos 0x0, ttl 50, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        183.14.209.53.43005 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 3211 7dd6 b70e d135  E..*..@.2.}....5
        0x0010:  dede dede a7fd 1a0d 0016 5872 3812 da35  ..........Xr8..5
        0x0020:  1638 0dd7 e400 0000 0000                 .8........
    19:28:42.219065 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 50)
        222.222.222.222.6669 > 183.14.209.53.43005: UDP, length 22
        0x0000:  4500 0032 0000 4000 4011 6fce dede dede  E..2..@.@.o.....
        0x0010:  b70e d135 1a0d a7fd 001e cb1c 2886 7656  ...5........(.vV
        0x0020:  36b3 2fe8 0501 0000 0000 12da 3516 380d  6./.........5.8.
        0x0030:  d7e4                                     ..
    19:29:13.506433 IP (tos 0x0, ttl 50, id 0, offset 0, flags [DF], proto UDP (17), length 42)
        183.14.209.53.60342 > 222.222.222.222.6669: UDP, length 14
        0x0000:  4500 002a 0000 4000 3211 7dd6 b70e d135  E..*..@.2.}....5
        0x0010:  dede dede ebb6 1a0d 0016 b882 386d 992b  ............8m.+
        0x0020:  fbe8 b60b f300 0000 0000                 ..........
    19:29:13.507141 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 54)
        222.222.222.222.6669 > 183.14.209.53.60342: UDP, length 26
        0x0000:  4500 0036 0000 4000 4011 6fca dede dede  E..6..@.@.o.....
        0x0010:  b70e d135 1a0d ebb6 0022 cb20 40f4 e0fe  ...5....."..@...
        0x0020:  55c3 e32a 2001 0000 0000 6d99 2bfb e8b6  U..*......m.+...
        0x0030:  0bf3 0000 0000                           ......
    # 可以看到服务器虽然有收到客户端的请求包并且也回应了，但是一直在循环这个过程，也就是说后面的包直接被 gfw 丢弃了

## 总结

基本上可以确定用 iptables 来丢弃 gfw 的干扰数据包是没有希望了，经过这次的折腾发现 TCPDUMP 其实挺好用的。

## 所用工具

* [TCPDUMP](http://www.tcpdump.org/)
* [Wireshark](http://www.wireshark.org/)

这两个都是抓包工具，Wireshark 用来分析，可以直接读取 TCPDUMP 输出的报文

## 参考文章及连接

<http://en.wikipedia.org/wiki/IPv4>
<http://en.wikipedia.org/wiki/User_Datagram_Protocol>
<http://danielmiessler.com/study/tcpdump/>
<http://blog.csdn.net/dog250/article/details/6647457>
