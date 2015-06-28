---
layout: post
title: "React-Native  起始"
description: ""
category:
tags: [React-Native]
---
{% include JB/setup %}

## 起始篇

### 要求

- 1. OS X -  目前我们只提供基于iOS(7+)的实现，并且Xcode只支持在Mac上运行。
- 推荐使用Xcode6.3及以上版本
- 推荐使用Homebrew来安装node，watchman和flow。
- `brew install node`。你是node或者npm新手吗？
- `brew instal watchman`。我们推荐你安装watchaman，否则你可能会踩到node自身watch功能的bug。
- `brew install flow`。如果你想使用flow的话。

### 快速上手

- `npm install -g react-native-cli`
- `react-native init AwesomeProject`

在新建的`AwesomeProject/`目录下

- 打开`AwesomeProject.xcodeproj`文件，并在Xcode里点击运行。
- 在你的编辑器里打开`index.ios.js`文件，编辑几行代码。
- 在iOS模拟器里，用`cmd+R`快捷键来重新加载app来查看你的改动。

恭喜你，你已经成功地运行和修改了你的第一个React Native app。

如果你在这期间碰到任何问题，参考排查问题页面。
