---
title: Rxswift share
date: 2018-01-09 22:31:32
tags:
---

原文：https://medium.com/@_achou/rxswift-share-vs-replay-vs-sharereplay-bea99ac42168

## RxSwift: share vs replay vc shareReplay

 对于使用RxSwift的初学者来说，经常会犯的错误是: 多次订阅同一个`observable`，却重复执行响应链

例如
 ```
 let results = query.rx.text
    .flatMapLatest { query in
        networkRequestAPI(query)
    }
results.subscribe(...)   // a network request
results.subscribe(...)   // another network request
 ```
 当同一个`observable`，需要被多次订阅的时候，而我们又不想这个`observable`对每个订阅者都执行。`RxSwift`为我们提供了几个操作符`share()`,`replay()`,`replayAll()`,`shareReplay()`,`publish()`,and `shareReplayLatestWhileConnected`来解决.那么我们应该使用哪一个呢？

 下面是关于这些操作符的对比

![Alt text](/img/RxSwift-share.png)

**Shared subscription:** 返回的可观察者共享源observable的一个底层订阅。 所有这些操作符都是如此。

**Connectable:** 在调用`connect()`函数之前，返回的`observable`不会发送事件。这允许多个订阅者在发送任何事件之前订阅。

**Reference counting:** 返回的observable的引用计数为订阅该observable的个数。当订阅者的个数从0到1时，订阅源观察值。当订阅者从0到1时，取消订阅和处理源观察。**注意：每次订阅者的数目从0到1改变的时候，observable都将被重新订阅**。
无论如何，同一时间订阅源observable的个数不会超过一个，并且所有并发订阅者将共享相同的订阅源observable。如果底层observable发送了完成或发送、错误的消息，则底层observable可能不会被重新订阅。这是一个灰色区域我建议尽量避免，可以通过确保引用计数下降到0时没有订户来完成。

**Replays events:** 在这些事件被发送后，操作符将重播事件给订阅`源observable`的订阅者。对于`replay(bufferSize)`和`shareReplay(bufferSize)`操作符，事件的个数最多为`bufferSize`。 对于`shareReplayLatestWhileConnected()`来说，最多重播一个事件。当订阅者的引用计数降为零时，重播事件的个数被清空。因此，使引用计数从0到1的订阅者不会收到重播事件。

如果你有`Subjects`，还可以使用`multicast`操作符。 但是这里描述的操作符是最流行的，并且能满足大部分使用场景。
