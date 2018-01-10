---
title: fastlane使用指南
date: 2017-07-31 22:36:46
tags:
---

在这篇教程里面，你讲学习如何使用fastlane将app部署到App Store。

**Note:**本教程大量使用了命令行，你不需要是一个精通终端者，但要了解一些命令行的基本使用。

学习本教程你需要了解code signing和iTunes Connect。如果你对这些不熟悉，请先阅读[这篇文章](https://www.raywenderlich.com/127936)

## 开始

下载[该教程Demo](https://koenig-media.raywenderlich.com/uploads/2016/12/mZone-swift3-starter.zip)，放到合适的路径

mZone，

fastlane 运行环境：

* OS X 10.9 (Mavericks)或者更新
* Ruby2.0或者更新版本
* 已付费的苹果开发者账号

因为fastlane是一系列Ruby scripts的集合，所以必须要匹配的Ruby环境。打开终端敲入以下命令可以查看当前Ruby版本
```
ruby -v
```
在终端里面敲入下面命令可以查看Xcode CLT是否安装
```
xcode-select --install
```
如果已经安装，你会得到这样的错误：` command line tools are already installed, use "Software Update" to install updates.`，否者，将为你安装Xcode CLT。

完成上面的操作后，就可以安装fastlane了。继续在终端里面敲入下面命令：
```
sudo gem install -n /usr/local/bin fastlane --verbose
```

## fastlane 工具链

* [produce](https://github.com/fastlane/fastlane/tree/master/produce)
在iTunes Connect和Apple Developer Portal中创建新的iOS应用程序。
* [cert](https://github.com/fastlane/fastlane/tree/master/cert)自动创建和维护签名证书。
* [sigh](https://github.com/fastlane/fastlane/tree/master/sigh)创建，更新，下载和修复配置文件。

## 设置fastlane
