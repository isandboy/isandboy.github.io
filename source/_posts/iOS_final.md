1. 如何解决 （IOS）UIWebview 内存泄漏

* 清理缓存
```
[[NSUserDefaults standardUserDefaults] setInteger:0 forKey:@"WebKitCacheModelPreferenceKey"]; 
```

* 退出当前页面时候，request一个空链接请求

* 换成wkwebview

2. 沙盒

* Documents
* Library 

   Caches [保存临时文件，"后续需要使用", 系统不会清理 cache 目录中的文件, 提供清理入口]

   Preferences [用户偏好，使用 NSUserDefault 直接读写]

* tmp [下次打开很有可能被自动删除了，这是临时文件夹]
* systemData


Caches文件夹不会在你备份手机数据的时候上传到iTunes上去，documents则会被备份上传

3. [扩大button点击区域](https://www.jianshu.com/p/06452c29a893)
4. [响应链]()
5. [数据库索引]()
6. [App做什么优化]()
7. Copy数组,里面元素是一样被copy吗

    不一定，因为如果数组里面的元素没有实现copy协议的话是不会被copy

8. [Https和http有啥区别](https://juejin.im/entry/58d7635e5c497d0057fae036)
9. Assign修饰字符串会有问题吗
10. Timer循环引用怎么情况
11. View和layer的关系
12. Block在arc和mrc区别
13. Assign修饰代理
14. arc和mar有啥区别

## 一、理解`属性`概念

@property: 编译器会自动生成实例变量和getter和setter方法

如果不希望自动生成存取方法和实例变量，那就要使用@dynamic关键字

属性特质有四类：

1. 原子性： 默认为atomic
    * nonatomic: 非原子性，读写时不加同步锁
    * atomic：原子性，读写时加同步锁
2. 读写权限： 默认为readwrite
    * readwrite: 可读可写
    * readonly: 只读
3. 内存管理
    * assign：对纯量类型做简单赋值操如 CGFlout, bool
    * weak: 弱拥有关系，设置方法既不保留新值，也不释放旧值，当指针指向的对象被销毁时，修饰的变量置为nil。#比如用来修饰delegate，避免循环引用问题，野指针问题
    * strong： 强拥有关系，保留新值，释放旧值
    * copy：拷贝拥有关系，设置方法不保留新值，将其拷贝
4. 方法名：
    * getter=: 指定get方法的方法名，常用
    * setter=：指定set方法的方法名，不常用

在iOS开发中，99.99..%的属性都会声明为nonatomic。
一是atomic会严重影响性能，
二是atomic只能保证读/写操作的过程是可靠的，并不能保证线程安全。

## 在对象内部尽量直接访问实例变量

1. 实例变量(_属性名)访问对象的场景
    * 在init和dealloc方法中，总是应该通过过访问实例变量读写数据
    * 没有重写getter/setter方法，也没有使用kvo
好处：不用走oc方法的派发机制，直接访问内存读写，速度快，效率高

2. 用存取方法访问对象的场景
    * 重写了gettter/setter方法
    * 使用了kvo监听值的变化

