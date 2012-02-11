---
layout: post
title: "MongoDB 使用小结"
category: 
tags: ['MongoDB']
---
{% include JB/setup %}

不知不觉的，使用 MongoDB 已经一年了，下面是根据官方文档以及自己的使用经验总结出来的

* 使用 ext4 或者 xfs 文件系统，系统当然是 64 位的 linux 系统，如果有条件的话使用 SSD 是很不错的选择
* 同传统关系型数据库一样，复杂的查询需要同时在多个字段上建立索引
* 关闭文件系统的 atime，在 `/etc/fstab` 中配置
* 不要使用大的 VM pages
* 字段名称尽可能简短，不同于关系型数据库，MongoDB 每个文档都需要保存字段名称
* 如果使用单台服务器，最好使用 Replication，另外也可以使用 1.8.0 版本，开启 journal 以免 crash 之后需要花费大量时间 repair 才能启动
* 如果需要根据时间排序，尽可能使用 _id 来排序，而不是加个时间的字段，除非是根据修改时间而不是插入的时间
* 调整系统文件描述符到 4k 以上
* 避免将某些关系型数据库的使用习惯带入 MongoDB，尽可能的将内容存成一个文档，而不是分拆成几个，即使是通过引用，同样是执行多次查询而不是一次，同时也需要注意单个文档最大为 4M，新版 MongoDB 增加到 16M
* 定时分析 MongoDB 日志，分析查询是否太慢，默认纪录超过 100ms 的查询
* 如果需要遍历数据，如建立 lucene 索引，一定要关闭 cursor 的 timeout，否则极有可能中途 cursor 超时导致任务执行失败
* 一些复杂的任务可以使用 MapReduce 来实现
* 关闭 httpinterface 如果你不使用的话
* 完整的备份使用 mongodump
* ObjectId 是根据时间戳、机器、及进程 pid 和自增组合的，不用担心会重复，当然除非你手动指定
* 要习惯使用 MongoDB 的 ObjectId 而不是使用类似 MySQL 的自增数字 id，除非有特殊要求
* 如果需要保存日志或者缓存之类的临时数据，使用 capped collection 以获得高性能

我这里大多数时间使用的是 MongoDB 1.6.3，运行于 Gentoo 和 Debian 环境，前者用于开发及自己折腾，后者用于生产环境。

备注：能力有限，如有不合理的地方敬请指正。