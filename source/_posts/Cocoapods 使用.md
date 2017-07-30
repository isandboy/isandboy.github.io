---
title: Cocoapods 使用
date: 2017-05-01 08:24:00
---

## 简介

CocoaPods是Swift和Objective-C项目的依赖管理器，它有超过2.6万个库并被超过160万个app使用， CocoaPods可以帮助您优化扩展项目。
CocoaPods管理的第三方开源库配置在https://github.com/CocoaPods/Specs 库中。

## 安装
```
//安装最新版本
sudo gem install cocoapods
//安装指定版本
sudo gem install cocoapods -v 1.1.1
```
NOTE：在这里给大家推荐一款pod版本管理工具`podenv`

地址：https://github.com/kylef-archive/podenv

## 卸载
```
sudo gem uninstall cocoapods
```

NOTE：一般国内用户安装失败，是因为ruby源问题。因为ruby的软件源rubygems.org，使用的是亚马逊的云服务，但被我天朝屏蔽了。

***一般的解决方式有两种：***
1. 移除https://rubygems.org 源，添加https://gems.ruby-china.org/ ，操作请看下面关于`gem sources` 命令介绍
2. 翻墙，关于如何翻墙问题，网上教程说的很详细，这里就不在累述。

作为一名开发者，推荐使用**第二种**方式

当cocoapods安装后，可以通过`pod --version`命令来查看当前安装的cocoapods版本

其他问题可参考：[安装CocoaPods遇到的问题及解决办法](安装CocoaPods遇到的问题及解决办法)

## 使用

1. 使用terminal切换到你的project目录
2. 创建一个Podfile文件，你可以使用`touch Podfile`创建，但是有一种更为优雅的方式，通过在terminal执行`pod init`命令
3. 在你的Podfile文件里面为所欲为吧，关于Podfile的配置，请参考cocoapods官方链接：https://guides.cocoapods.org/syntax/podfile.html
4. 保存Podfile文件，运行`pod install`
5. 以后打开你的项目都使用**ProjectName.xcworkspace**而非**ProjectName.xproject**

## pod install vs pod update

`pod install`和`pod update`命令，也许大家只会在第一次使用cocoapods配置project的时候使用一次`pod install`，甚至一次都没用过，一般都直接使用`pod update`命令了

但是：`pod install`和`pod update`有什么区别呢，分别在什么时候使用呢？

***NOTE：***

* install一个新的pods在你的project中，即使在你的project里面已经有一个**Podfile**文件或者之前已经执行了`pod install`命令
* 使用`pod update [PODNAME]`，仅仅当你想要更新pods到一个新的版本的时候

`pod update`会更新所有Podfile文件里面所指定的依赖到一个新的版本。

`pod update [PODNAME]`仅仅会更新**[PODNAME]** 库

***pod install***

它不仅适用于当你第一次为你的project去检索新的pods的时候，也适用于你每次编辑`Podfile`文件（如：添加，更新，或者移除一个pod的时候）

但是有以下需要注意：

* 每次执行`pod install`命令，会根据**Podfile.lock**文件已经明确列出的库以及对应的版本去下载并安装pods，
而不会去检查相应的库是否有新的版本。其中**Podfile.lock**文件会记录已经引入的pod库和相应的版本。
* 当你pods没有在**Podfile.lock**文件列出的时候，它会去根据Podfile文件里面所描述的依赖去下载（如：pod 'MyPod', '~> 1.2'）

NOTE:

***pod outdated***

当你使用`pod outdated`命令时，Cocoapods会列出比Podfile.lock文件中的pod更新的所有pods。这个时候你就可以使用`pod update PODNAME1 PODNAME2`去更新PODNAME1和PODNAME2的版本（pod update 后面可以跟多个pod）

***pod update***

当你执行`pod udpate PODNAME`命令的时候，它将会更新该库到一个满足在Podfile文件里面对该库限制的一个最新版本，而不会关心Podfile.lock文件里面对该库有没有限制

***pod install/update [--no-repo-update]***

执行`pod install/update` 命令会首先更新本地的配置库

执行`pod install/update --no-repo-update` 命令不会更新本地的配置库

本地配置库位置可以通过以下命令查看
```
cd ~/.cocoapods/repos
ls
```

关于Cocoapods的使用更加详细的介绍：可以参考 **最新的 CocoaPods 的使用教程** 系列文章介绍
http://ios.jobbole.com/90957/

**master**文件夹是官方的open sources library


***gem sources***

关于查看当前电脑的ruby源，有如下一些命令需要了解

```
gem sources -l #查看当前已经制定的源
gem sources -r SOURCE_URI #移除源
gem sources -a SOURCE_URI #添加源
```
详情请参考：`gem sources --help` 来查看
