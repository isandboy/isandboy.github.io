---
title: SYPayKit简介
date: 2017-07-30 16:12:25
tags:
---

## 问题

一款产品，随着版本的迭代，难免会加入支付功能。而现在的主流支付方式，主要有支付宝、微信、银联支付、苹果银联支付等。
每家支付SDK提供的对外接口不统一，会引起使用起来的混乱，以及维护成本的增加。
比如，在支付成功后由第三方app回调回来：
```
- (BOOL)application:(UIApplication *)application openURL:(nonnull NSURL *)url options:(nonnull NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    if ([[url absoluteString] rangeOfString:@"wx"].location != NSNotFound &&
               [[url absoluteString] rangeOfString:@"pay"].location != NSNotFound) {
        return [WXApi handleOpenURL:url delegate:self];
    } else {
        /*其他支付方式的处理*/
        if ([url.host isEqualToString:@"safepay"]) {
            [[AlipaySDK defaultService] processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) {
                /*处理resultDic*/
            }];
            return YES;
        }
        if ([url.host isEqualToString:@"platformapi"]) {//支付宝钱包快登授权返回authCode
            [[AlipaySDK defaultService] processAuthResult:url standbyCallback:^(NSDictionary *resultDic) {
                /*处理resultDic*/
            }];
            return YES;
        }
    }
    return YES;
}
```
接口抽象统一后：
```
- (BOOL)application:(UIApplication *)application openURL:(nonnull NSURL *)url options:(nonnull NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    return [SYPay handleOpenURL:url sourceApplication:nil];
}
```

## 支付流程

1. 进入支付页面
2. 选择支付方式
3. 根据当前支付方式去后台拉取订单，拉取成功后调起当前支付方式对应的app/走网页（如未安装支付宝客户端，银联等）
4. 在第三方app完成支付后，回调回来，去后台验证订单是否完成支付

NOTE：目前一般的通用做法是支付页面是网页，由网页发起拉取订单的请求，订单部分是后台处理好的订单（web端和native端都不做任何处理），然后web端再把订单通过js交互告诉native端，再由native端调用第三方支付SDK发起支付，支付成功后再回调给web端。

关于支付结果验证部分，目前我们这边是在web端去处理的，native端只负责调用第三方支付SDK，回调支付结果给web端

## 思路

<!-- 首先参考了 [Ping++聚合支付iOS接口设计](https://github.com/PingPlusPlus/pingpp-ios)

做法上他们把各种支付方式的 -->

当去新加入/移除一种支付方式，不会对抽象出来的接口产生影响，也就是说不用做任何修改。

所以就抽象出了一套协议，当新加入一种支付方式时候，只需要实现那套协议即可。

目前的做法是：
1. 生成某种支付方式的实例
```
SYUnionPay *alipay = [[SYUnionPay alloc] init];
```
2. 生成订单
```
NSDictionary *order = @{kMSPayOrderKey:@"201506221028315777129"};
```
3. 发起支付
```
[SYPay payment:payment withOrderInfo:order withCompletion:^(SYPayResultStatus status, NSDictionary * _Nullable returnedInfo, NSError * _Nullable error) {
    NSLog(@"success:%d\n,resultDic:%@\n,error:%@", success, returnedInfo, [error localizedDescription]);
}];
```

NOTE: 有的做法是把支付方式，放到生成的订单字典里面，由此去区分是哪种支付方式如：
```
NSDictionary *order = @{kMSPayOrderKey:@"201506221028315777129", @"payType": @"alipay"};

```

## 使用

安装：目前还处于测试阶段，所以版本还未固定
```
pod "SYPayKit", :git => 'https://github.com/isandboy/SYPayKit'
```
默认包含支付宝、微信、银联支付

目前支持以下几种支付方式

Alipay （支付宝）

WXPay（微信）

UnionPay（银联）

```
pod 'SYPayKit/Alipay'
pod 'SYPayKit/WXPay'
pod 'SYPayKit/UnionPay'
```
调用部分
```
SYUnionPay *alipay = [[SYUnionPay alloc] init];
NSDictionary *order = @{kMSPayOrderKey:@"201506221028315777129"};
[SYPay payment:payment withOrderInfo:nil withCompletion:^(SYPayResultStatus status, NSDictionary * _Nullable returnedInfo, NSError * _Nullable error) {
    NSLog(@"success:%d\n,resultDic:%@\n,error:%@", success, returnedInfo, [error localizedDescription]);
}];

```

接口参数说明：
```
/**
 *  支付调用接口
 *
 *  @param charge           以kSYPayOrderKey为key的订单，对应的value为格式参考readme说明文档
 *  @param viewController   银联渠道需要
 *  @param completionBlock  支付结果回调 Block
 */
+ (void)payment:(SYPayment *)payment withOrderInfo:(NSDictionary *)charge viewController:(nullable UIViewController *)viewController withCompletion:(SYPayResultHandle)handle;
```

1. 其中payment是对应的支付方式的实例
2. charge参数需要和后台约定成以下格式

```
# 支付宝支付order
{
		kSYPayOrderKey: "partner=\"--------------\"&seller_id=\"-------------\"&out_trade_no=\"-----------\"&subject=\"areyouok\"&body=\"nama\"&total_fee=\"0.01\"&notify_url=\""&service=\"\"&payment_type=\"1\"&_input_charset=\"utf-8\"&it_b_pay=\"30m\"&sign=\"GsSZgPloF1vn52XAItRAldwQAbzIgkDyByCxMfTZG%2FMapRoyrNIJo4U1LUGjHp6gdBZ7U8jA1kljLPqkeGv8MZigd3kH25V0UK3Jc3C94Ngxm5S%2Fz5QsNr6wnqNY9sx%2Bw6DqNdEQnnks7PKvvU0zgsynip50lAhJmflmfHvp%2Bgk%3D\"&sign_type=\"RSA\"&appenv=\"system= ^version=\"&goods_type=\"0\"&rn_check=\"F\""
  }

# 微信支付order

{ kSYPayOrderKey: {
			"appid": "wx-----------",
			"partnerid": "-----------",
			"noncestr": "-----------",
			"prepayid": "wx-----------",
			"packagevalue": "Sign=WXPay",
			"timestamp": "-----------",
			"sign": "-----------"
		}
}

# 银联支付order
{kSYPayOrderKey:@"--------------------" }
```
更详细的情况可以参考demo里面的SYPayManager类

## 其他

iOS 9 以上版本如果需要使用支付宝和微信渠道，需要在 Info.plist 添加以下代码：
```
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>weixin</string>
    <string>wechat</string>
    <string>alipay</string>
</array>
```

如果项目里面还用到了支付宝、微信、银联支付的SDK，请使用
```
pod 'WechatOpenSDK'
pod 'SYUPPaySDK'
pod 'SYAlipaySDK'

```
否则会出现和该库冲突的问题。这三个SDK里面分别对应集成了相应的官方SDK，并且支持打包成动态库。

该库可以用于Objective-C、swift工程，并支持打包成动态库。
