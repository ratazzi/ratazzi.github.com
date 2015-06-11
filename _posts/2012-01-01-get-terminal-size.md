---
layout: post
title: "获取终端大小"
category: 
tags: ['shell', 'Gentoo', '终端大小']
---

用过 Gentoo 的同学一定会觉得 Gentoo 下的 portage 等工具以及启动脚本输出非常赞，不仅仅是颜色那么简单，还用到了终端的大小，下面是几种获取终端大小的方法。

`Python`
	#!/usr/bin/env python
	# encoding=utf-8
	import curses
		
	if __name__ == '__main__':
		curses.setupterm()
		print curses.tigetnum('lines')
		print curses.tigetnum('cols')
				
`Shell`
	#!/bin/bash
	echo $LINES
	echo $COLUMNS
		
	# or
		
	tput lines
	tput cols
		
	# or
		
	stty size
