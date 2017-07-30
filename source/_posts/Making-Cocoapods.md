---
title: Cocoapods创建私有库
date: 2017-05-01 08:24:00
---

这篇文章主要是介绍怎么去创建和维护一个Cocoapods

1. Using Pod Lib Create
2. Getting setup with Trunk

## Getting setup with Trunk

使用Pod创建库，打开终端敲入以下命令：

```
pod lib create MyLirary
```

NOTE: 如果你想要使用你自己的**pod-template**去创建Pod库，你可以在上面的命令后面追加`--template-url=url`参数，其中url是你的git repo库地址。
如果不追加参数默认拉取的模板地址是https://github.com/CocoaPods/pod-template

敲入以上命令回车后，会让你回答几个问题，按照提示填入就行。

创建Project之后会，会自动运行`pod install`命令，让我们来看看Cocoapods给我们生成的文件目录结构

```
$ tree MyLib -L 2

  MyLib
  ├── .travis.yml
  ├── _Pods.xcproject
  ├── Example
  │   ├── MyLib
  │   ├── MyLib.xcodeproj
  │   ├── MyLib.xcworkspace
  │   ├── Podfile
  │   ├── Podfile.lock
  │   ├── Pods
  │   └── Tests
  ├── LICENSE
  ├── MyLib.podspec
  ├── Pod
  │   ├── Assets
  │   └── Classes
  │     └── RemoveMe.[swift/m]
  └── README.md
```
* .travis.yml - 一个travis-ci配置文件.
* _Pods.xcproject - 支持Carthage的对你Pod's Project的链接.
* LICENSE - 默认是 MIT License.
* MyLib.podspec - 你的Library的配置文件.
* README.md - a default README in markdown.
* RemoveMe.swift/m - 无用文件删除即可.
* Pod - 放置你的库代码和资源的文件夹
* Example - Demo，方便你去调试你的库

## Getting setup with Trunk

关于怎么发布自己的Pods到Cocoapods trunk，

请参考：
1. http://www.tuicool.com/articles/6FF7fi
2. https://guides.cocoapods.org/making/getting-setup-with-trunk.html

说明：关于 `.podspec` 文件的写法说明也可以参考[CocoaPods学习系列4——进阶用法](http://www.cnblogs.com/ALongWay/p/6541289.html)

## Private Pods

***1. Create a Private Spec Repo***
参考：https://guides.cocoapods.org/making/private-cocoapods.html

## 私有远端仓库
```

```
##

待修改

## 验证podspec有效性

当开发完私有库之后，把配置库，提交到私有创库，或者cocoapods的公有库仓库，流程如下

***1. 本地验证仓库有效性***

```
pod lib lint --source --use-libraries --allow-warnings --verbose
```
其中pod lib lint 之后的参数都是可选参数

* --source：当开发私有库的时候，
* --use-libraries: podspec文件里面通过pod dependencies指定的依赖，是否包含静态库，当不加此参数会验证失败。因为swift开发的库，不能包含静态库，所以此参数只针对Objective-C
* --allow-warnings：是否忽略警告。
* --verbose：详细log


***2. 提交代码***

```
git add .
git ci -m"edit commit message"
git tag 0.1.0
git push origin master tags
```

***3. 推送podspec配置文件到私有库***

```
pod repo push SYSpecs [MyLib.podspec] --use-libraries --allow-warnings --verbose
```
其中参数与验证参数一个含义，验证的时候加了哪些可选参数，推送配置文件到私有库的时候也要加
