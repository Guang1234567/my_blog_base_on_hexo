---
title: 基于 nio2 的 swift coroutine 库
tags:
  - Coroutine
  - 协程
  - NIO
  - Swift
  - iOS
  - Android
  - macOS
  - Windows
  - Swift-Server
  - CrossPlatform
copyright: true
sticky: false
hide: false
cover: /2020/08/10/swift-coroutine-nio2/Swift_With_Android.png
top_img: /2020/08/10/swift-coroutine-nio2/Swift_With_Android.png
categories:
  - [Swift]
  - [Coroutine]
  - [NIO]
keywords: 'Coroutine,Swift,CrossPlatform'
description: 一个用 Swift 和汇编语言编写的协程库
katex: false
abbrlink: 8161a164
date: 2020-08-10 12:08:09
---


## 适用系统平台

- Android
- iOS
- macOS
- Linux
- Windows

基本上主要的系统平台都支持 -_-||.

本意只是想给 Android 与 iOS 用 `Swift5` 和 [apple/swift-nio](https://github.com/apple/swift-nio) 写一套跨平台可复用的 `IM` 代码 (支持 xmpp), 

另外加上 `Coroutine` 这一特性支持.

结果后端编程中常用的 (Linux) 也顺便支持了 -_-||.

## 背景

>SwiftNIO is a cross-platform asynchronous event-driven network application framework for rapid development of maintainable high performance protocol servers & clients.
>
>It's like [Netty](https://netty.io/), but written for Swift.

SwfitNIO 是一款基于事件驱动的跨平台网络应用程序开发框架，其目标是帮助开发者快速开发出高性能且易于维护的服务器端和客户端应用协议。

这篇文章主要介绍下面这个库的使用方法:

[Swift_Coroutine_NIO2](https://github.com/Guang1234567/Swift_Coroutine_NIO2) 

——   一个能为 [apple/swift-nio](https://github.com/apple/swift-nio) 增添 `Coroutine`( 协程 ) 功能的库, 另外它是用最新的 Swift5 编写的.

## 谁能无缝集成 [Swift_Coroutine_NIO2](https://github.com/Guang1234567/Swift_Coroutine_NIO2) ?

使用 `Swift` 和 `Rust` 进行后端编程是最近一两年比较热门的话题, 一些 `WebFramework` 应运而生.
其中 `swift-server-side` 的 `WebFramework` 相对比较热门的有:

- [vapor/vapor](https://github.com/vapor/vapor)
- [IBM-Swift/Kitura](https://github.com/IBM-Swift/Kitura)
- [NozeIO/MicroExpress](https://github.com/NozeIO/MicroExpress) 
    - Node like web framework using NIO. 
    - [Tutorial](https://www.alwaysrightinstitute.com/microexpress-nio2/): How to write a Swift HTTP endpoint [Express.js](http://expressjs.com/en/starter/hello-world.html)-like, using middleware and routing

它们都是基于[apple/swift-nio](https://github.com/apple/swift-nio). 

而 [Swift_Coroutine_NIO2](https://github.com/Guang1234567/Swift_Coroutine_NIO2) 是一个能为 [apple/swift-nio](https://github.com/apple/swift-nio) 增添 `Coroutine`( 协程 ) 功能的库

**故凡是基于 [apple/swift-nio](https://github.com/apple/swift-nio) 的 `WebFramework`, [Swift_Coroutine_NIO2](https://github.com/Guang1234567/Swift_Coroutine_NIO2) 都支持 !!!**

## 用法

下面将介绍如何在一些比较热门的 `swift-webframework` 中使用 [Swift_Coroutine_NIO2](https://github.com/Guang1234567/Swift_Coroutine_NIO2) .

### Usage in vapor

- 例子程序: 

[vapor4_auth_template](https://github.com/Guang1234567/vapor4_auth_template/blob/e8c33a489f3661e22fc9cf3faaa6920f4527fad0/Sources/App/Controllers/UserController.swift#L80-L135)

- 先来看看 `login` 方法原来的样子. 使用 `EventLoopFuture Chain` 避免了 `Callback Hell`. 符合人类思维方式的链式写法.

```swift
    func login(req: Request) throws -> EventLoopFuture<UserToken> {
        let user = try req.auth.require(User.self)
        let userTokenBeDeleted = req.auth.get(UserToken.self)

        return req.application.threadPool.runIfActive(eventLoop: req.eventLoop) {
            try user.generateToken()
        }.flatMap { token in
            let saveToken = token.save(on: req.db)
                                 .map {
                                     token
                                 }

            if let userTokenBeDeleted = userTokenBeDeleted {
                return UserToken.query(on: req.db)
                                .filter(\.$value == userTokenBeDeleted.value)
                                .delete(force: true)
                                .flatMap {
                                    saveToken
                                }
            } else {
                return saveToken
            }
        }
    }
```

但 `saveToken` 遇到类似于下面截图所示的情况, 工作流程中后续的多个步骤都需要前面的 `saveToken` 的结果

![](https://user-gold-cdn.xitu.io/2020/7/16/173554db108ed53d?w=884&h=299&f=jpeg&s=48826)

> Picture is copied from https://arrow-kt.io/docs/0.9/patterns/monads/

那上面截图这种情况如何解决? 使用 `async await` 形式, 但 `Swift` 语言(编译器)没有内置这一特性(语法糖).

编写 [Swift_Coroutine_NIO2](https://github.com/Guang1234567/Swift_Coroutine_NIO2) 的其中一个初衷就是为了添加这一特性 `async await`, 突破编译器的限制 !

接着看看与上面截图相关的采用 `async await` 形式的伪代码:

```swift
func loadSpeaker() -> CoFuture<Speaker> {
    return CoFuture(name, DispatchQueue.IO) { (co: Coroutine) in
        // running on  "IO"  thread
    }
}

func nextTalk() -> CoFuture<Talk> {
    return CoFuture(name, DispatchQueue.Single) { (co: Coroutine) in
        // running on  "Single"  thread
    }
}

func getConference() -> CoFuture<Conference> {
    return CoFuture(name, DispatchQueue.Net) { (co: Coroutine) in
       // running on  "Net"  thread
    }
}

func getCity() -> CoFuture<City> {
    return CoFuture(name, DispatchQueue.main) { (co: Coroutine) in
       // running on  "Main"  thread
    }
}
```

```swift
func workflow() throws -> Void {
    let speaker = try repository.loadSpeaker().await()
    let talk = try speaker.nextTalk().await()
    let conference = try talk.getConference().await()
    let city = try conference.getCity().await()

    // needs both Speaker and City objects
    reservations.bookFlight(speaker, city).await()
}
```

好像可读性更好了!

- 基于 [Swift_Coroutine_NIO2](https://github.com/Guang1234567/Swift_Coroutine_NIO2) 重新改写 `login` 方法后的样子.

```swift
func login(req: Request) throws -> EventLoopFuture<UserToken> {

        let user = try req.auth.require(User.self)
        let userTokenBeDeleted = req.auth.get(UserToken.self)

        let eventLoop = req.eventLoop
        let ioThreadPool = req.application.threadPool
        return EventLoopFuture<UserToken>.coroutine(eventLoop) { co in

            try co.continueOn(ioThreadPool)

            let token = try user.generateToken()

            try co.continueOn(eventLoop)

            if let userTokenBeDeleted = userTokenBeDeleted {
                try UserToken.query(on: req.db)
                             .filter(\.$value == userTokenBeDeleted.value)
                             .delete(force: true)
                             .await(co)
            }

            return try token.save(on: req.db)
                            .map {
                                token
                            }
                            .await(co)
        }
    }
```

- 代码解析

    - 创建一个 Coroutine
    
    ```swift
    EventLoopFuture<UserToken>.coroutine(eventLoop) { co in
    // other code ...
    }
    ```
    
    - 线程随意切换, 真是一个 diao 到 💥 的功能!
    
    ```swift
    let eventLoop = req.eventLoop
    let ioThreadPool = req.application.threadPool
    
    // switch to io thread for some expensive work
    // ----------- 
    try co.continueOn(ioThreadPool)
    
    // some expensive work here ...
    
    // switch back to the request's eventloop
    // ----------- 
    try co.continueOn(eventLoop)
    ```
    
    - 提供 `await` "语法糖"
    
    ```swift
    let f: EventLoopFuture<UserToken> = token.save(on: req.db).map { token }.await(co)
    let token: UserToken = try f.await(co)
    retrun token
    ```