---
layout: post
title: "重装 OS X 后尽可能的自动化"
category:
tags: []
---
{% include JB/setup %}

每次重装完 OS X 之后的设置过程还是很不大愉快的，于是我决定将一些基本设置尽可能的自动化，就像这样:

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

我非常喜欢这种方式，直接使用系统自带工具完成。

主要的设置：

* 基本的键盘、鼠标、Trackpad 设置等
* Finder
* 共享
* 安装命令行工具及应用程序
* Dock 位置、大小及图标
* 输入法
* 还有一堆 Dropbox 的目录或者文件需要软链接

首先最重要的升级系统及安装 Xcode，一些网上搜索到的设置我不做解释，很多人都知道 OS X 的设置基本上都是基于 plist 的，命令行工具 `defaults` 就是用来访问这些 `plist` 文件的。当然还有一件非常重要的事情就是能够顺利的接入*互联网*，否则整个过程将会非常的不愉快。

Finder 的默认设置真的是很不适合程序员，特别是 All My Files 这个功能真的是完全没有用处，我需要定制 Sidebar，但是搜索了很多资料发现用 `defaults` 是做不到的，甚至有人用 `Objective-C` 写了个命令行工具来完成这件事，但我用的是另外一种方式 `Apple Script`，最终的脚本是需要图形界面交互的

	tell application "Finder"
        activate
    end tell

    tell application "System Events" to keystroke "," using {command down}

    tell application "System Events"
        tell process "Finder"
            tell window "Finder Preferences"
                click button "Sidebar" of toolbar 1
                # delay 0.5
                tell checkbox "Movies" to if value is 0 then click
                tell checkbox "Music" to if value is 0 then click
                tell checkbox "Pictures" to if value is 0 then click
                tell checkbox "ratazzi" to if value is 0 then click
            end tell
        end tell
    end tell

    tell application "Finder" to close window "Finder Preferences"

在写这个脚本的过程中，用到了一个很重要的工具，`Accessibility Inspector.app` 在 `Xcode.app/Contents/Applications` 可以找的到，将鼠标移到需要勾选的 `checkbox` 你会看到 `Accessibility Inspector.app` 中显示为 `AXCheckBox`，其中 `AXTitle` 如果为空的话就不能使用 `checkbox "ratazzi"` 这样的方式定位了，后面会讲到另外一种方式

设置 Finder 新窗口默认的路径

	# Use current directory as default search scope in Finder
	defaults write com.apple.finder FXDefaultSearchScope -string "SCcf"
	# Open new windows on the user's home
	defaults write com.apple.finder NewWindowTarget -string "PfHm"
	defaults write com.apple.finder NewWindowTargetPath -string "file://$HOME"
	
很遗憾的是 Trackpad 的设置也不能完全通过 `defaults` 做到，同样使用 `Apple Script` 来完成，这里就用到了另外一种定位 `checkbox` 的方法

	# 注意是从 1 开始的，这里是开启三个手指拖动
	tell checkbox 4 to if value is 0 then click

最复杂的应该是打开共享目录了，这个没有 `Accessibility Inspector.app` 的帮助差不多是不可能完成的，其实我是照着网上改的，原始的脚本是打开 AirPort 共享的，不过是针对很老的 OS X 版本了。

Dock 的设置都可以通过 `defaults` 完成，不过我写了一个更复杂的脚本，用 `homebrew cask` 来安装我所需要的 App，然后按照我指定的顺序放到 Dock 上，这也是最酷的一步了。

然后是输入法，我使用 `Squirrel`，将配置生成后需要部署一下，但是没有命令行工具，最后也是借助 `Apple Script` 实现。

最后的效果

	curl -fsSL https://example.com/snippets/bootstrap.sh | bash

整个脚本是可以重复执行的，想要整个过程一次性完成的话前提是要有非常好的网络，另外脚本中某些行故意写成很长的行并且加了 `#trackpad` 是为了重复执行时可以很方便的通过 `sed` 来过滤，因为涉及到图形界面交互的一次就可以完成了。

[https://gist.github.com/ratazzi/91fb1a75c19f707cadb9](https://gist.github.com/ratazzi/91fb1a75c19f707cadb9)