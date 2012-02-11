---
layout: post
title: "Ubuntu 10.04 下安装 PyLucene 3.0.1"
category: 
tags: ['Ubuntu', 'pylucene', 'lucene', 'lucid', 'python']
---
{% include JB/setup %}

环境：
	* Ubuntu 10.04 64bit
	* Python 2.6.5
	* Ant 1.8.0
	* setuptools 0.6c11
	* jdk 1.6.0_20

首先保证上面的软件包都安装好，版本不一定要和我一样，另外可能需要安装 `python-2.6-dev`，否则会提示找不到 `Python.h`，然后就可以开始安装了
	`tar xzvf pylucene-3.0.1-1-src.tar.gz && cd pylucene-3.0.1-1/jcc`
	
需要编辑 `setup.py`
	JDK = {
		..
		'linux2': '/usr/local/jdk', # 这个目录比需要有 include 目录及相关头文件
		..
	}
	
然后安装 `jcc`，build 过程可能会有错误，只需要执行输出的命令后再 build 即可
	python setup.py build && sudo python setup.py install
	cd ..
	
编辑 `Makefile`，把适合你系统那一块注释删掉
	PREFIX_PYTHON=/usr
	ANT=ant
	PYTHON=$(PREFIX_PYTHON)/bin/python
	# 这里主要根据 Python 版本和 jcc 的安装决定
	JCC=$(PYTHON) -m jcc.main --shared # --shared 一般不需要加
	NUM_FILES=2

最后安装 `Pylucene`，这个过程会调用 ant 编译 `Lucene Java`
	make && sudo make install
