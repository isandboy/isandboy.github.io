---
title: dynamic framwork
date: 2017-07-28 23:42:27
tags:
---

## 问题

在swift工程里面，如果是通过pod去管理公有库，或者团队里面的私有库。需要在`Podfile`文件里面加入`use_frameworks!`

但一般的工程里面都要依赖于支付宝、微信等第三方SDK，有些第三方SDK是静态库。
如果是通过Pod 直接引入该静态库，如果只是一级依赖，`pod install/update` 是可以通过，如果是有多级依赖，执行上面命令就会出现如下错误信息
```
[!] The 'xxxxx' target has transitive dependencies that include static binaries:
```
正确打包出来的动态库，应该像这样。打开Product/xxxx.app -> Frameworks 文件夹，如下图所示:

![Alt text](/img/dynamic01.jpeg)

## 原理见参考

## 实战

1. 通过`pod lib create LibraryName` 命令创建
2. 把静态库放到你开发的库里面
3. 如果当前开发的库里面，除了静态库/动态库（假的）和头文件外，没有其他文件，你需要随便创建个类，否则不行，如果有其他方式一块交流
4. 关于如果`.podspec`文件的写法，可以参考:https://github.com/isandboy/SYAlipaySDK

## 注意

当没有使用到第三方静态库中的相关API时，链接器帮我们链接动态库的时候可能并不会把静态库吸附进来。我们手动在build Setting的other link flags加上-all_load标记

如果是通过 `pod lib create` 命令去创建模块，可以在`.podspec` 文件里面加入如下命令
```
spec.pod_target_xcconfig = { 'OTHER_LDFLAGS' => '-all_load' }
```

## 参考：
[组件化-动态库实战](http://www.cocoachina.com/ios/20170427/19136.html)
