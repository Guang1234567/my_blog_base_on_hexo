---
title: 使用 Swift 从零开始编写一个跨 Android iOS Linux macOS Windows 平台的协程库 (Coroutine)
tags:
  - Coroutine
  - 协程
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
cover: /2020/08/10/swift-coroutine/Swift_With_Android.png
top_img: /2020/08/10/swift-coroutine/Swift_With_Android.png
categories:
  - [Swift]
  - [Coroutine]
keywords: 'Coroutine,Swift,CrossPlatform'
description: 一个用 Swift 和汇编语言编写的协程库
katex: false
abbrlink: 16dc4330
date: 2020-08-10 11:54:47
---

## Overview

单线程编程是简单的.

假设我们所处世界的工艺水平达到匪夷所思的境界,那么就能制作出一颗单核的超级 CPU. 无论面对多么复杂的运算过程, 它的运算速度总是快到人类无法感知有任何时间上的停顿.

同时这也是软件工作者最想发生的事件之一: 因为挣脱了硬件上的桎梏,不用为之在软件编码上做出妥协!


回到现实, CPU 的纵向发展速度已经逐步开始放缓, 既然短时间内质变不了,那么先来量变吧, 于是多核的 CPU 面世, 并行计算成为现实.

并行计算的出现给软件工作者带来一系列的问题, 但这些问题本质都是围绕着"协作"这个核心问题展开的. 


举个工厂工人的例子: 

要安排若干个工人手工组装一辆劳斯莱斯, 但组装到一半的时候发现材料不够, 此时有两种解决办法:

1. 工人停下手头上的事情, 什么也不做, 原地待命. 直到足够的材料被采购回来, 通知他们继续干活.

2. 工人停下手头上的事情, 但上级工作人员根据实际情况协调, 让他们先干别的工作,直到足够的材料被采购回来.
    
这个例子形象地解释了 `阻塞` (工人停下来,什么也不做,原地待命) 与 `非阻塞` (工人被安排做其他事情).
   
其实 `协程 Coroutine` 就是在软件领域中一些实现 `非阻塞` 的技术的统称.

不同编程语言不同平台分别有各种各样的具体实现. 

此文着重于结合使用 Swift 和 汇编语言来实现 `协程 Coroutine` 库, 得益于 Swift 和 C 的良好兼容, 此库也能被 C 所调用.

接下来要探讨如何使用 Swift 和 汇编语言来实现 `协程 Coroutine`的一些技术细节.
    

## 技术细节一:  Boost.Context 库

先别着急, 先看看一个最简单的场景 (单线程) :

有 1 个线程 (thread_01), 3 个函数 (f1 f2 f3).

假设这 3 个函数分别代表着需要完成的 3 项不同的独立(无依赖关系)的任务, 这 1 个线程代表着执行者, 这个执行者要去完成这 3 项任务. 

先通过伪代码展示一下这个执行过程:

```
thread_01.run {
    f1()
    f2()
    f3()
}
```

接着挑战来了, 假设运行于 thread_01 上的 f1 运行到函数体一半的时候,发现某些条件不满足就不能继续往下执行, 此时为了解放 CPU 的运算能力, 让 thread_01 跳转到 f2, 同样 f2 运行到一半时遇到同 f1 一样的状况,所以让 thread_01 再从 f2 跳到 f3 去执行.


f3 顺利执行完毕跳回 f2 的"挂起点", 接着 f2 执行完毕跳回 f1 的"挂起点", f1 继续往下执行.


为了完成上面的挑战, 一般的解决方案肯定是用 C 语言的 `goto` 和代表着"挂起点"的 `label` 来实现上面所述的一系列无条件跳转.

回到现实, 在程序运行过程中, 你永远不知道有多少个任务, 你不可能在编译时静态地为不明数量的任务指定 `label`.

> Note:
>
> 其实单纯用 C 也能实现单线程版的协程,
> 想到这个办法的瑞典人真是个天才,已经对汇编的理解达到炉火纯青的地步.
>
> 下面是与之相关的中文文章:
>
> https://coolshell.cn/articles/10975.html

那么别的语言是怎么实现的? 

像 Golang 和 Kotlin-Coroutines 之流都是通过编译技术来实现类似于 `goto` 这种跳转, lua 不太清楚, 基本都是在语言层面就已经实现.


像 Swift 和 C 和 C++ 和 Rust 就需要直接 or 间接地借助汇编语言来实现 `跳转`的功能.
其中 `Boost.Context` 就是这么一个高效且被广泛使用的跨多平台的 C++ 库, 其核心 `跳转` 功能就是用汇编语言来编写.


本文余下部分将要编写的 `Swift_Coroutine` 库也是基于 `Boost.Context` 的核心部分, 由于只需要核心部分, 因此基于模块化思想把核心功能对应的汇编代码拷贝出来组织成一个单独的 Swift 库:

[Swift_Boost_Context](https://github.com/Guang1234567/Swift_Boost_Context)

用法:

```swift
import Swift_Boost_Context

func f1(_ data: Int) throws -> String {
    defer {
        print("f1 finish")
    }
    print("main ----> f1  data = \(data)")

    let bc2: BoostContext = makeBoostContext(f2)
    print("bc2 = \(bc2)")
    let resultF2ToF1: BoostTransfer<String> = try bc2.jump(data: "I am f1")
    print("f1 <---- f2   resultF2ToF1 = \(resultF2ToF1)")

    return "7654321"
}

func f2(_ data: String) throws -> String {
    defer {
        print("f2 finish")
    }
    print("f1 ----> f2  data = \(data)")

    return "1234567"
}

func main() throws {
    let bc1: BoostContext = makeBoostContext(f1)
    print("bc1 = \(bc1)")
    //let bc2: BoostContext = makeBoostContext(f2)
    //print("bc2 = \(bc2)")

    let resultF1ToMain: BoostTransfer<String> = try bc1.jump(data: 123)
    print("main <---- f1 resultF1ToMain = \(resultF1ToMain.data)")
    //let _: BoostTransfer<String> = try resultF1ToMain.fromContext.jump(data: 123)
}

do {
    try main()
} catch {
    print("main : \(error)")
}
```

控制台输出:

```ruby
bc1 = BoostContextImpl(_spSize: 65536, _sp: 0x00007fec89500000, _fctx: 0x00007fec894fffc0)
main ----> f1  data = 123
bc2 = BoostContextImpl(_spSize: 65536, _sp: 0x00007fec89511000, _fctx: 0x00007fec89510fc0)
f1 ----> f2  data = I am f1
f2 finish
f1 <---- f2   resultF2ToF1 = BoostTransfer(fromContext: BoostContextProxy(_fctx: 0x00007fec89510eb0))
f1 finish
main <---- f1 resultF1ToMain = 7654321

Process finished with exit code 0
```

### 解锁 Boost.Context 更多的姿势

[Swift_Boost_Context](https://github.com/Guang1234567/Swift_Boost_Context) 只能用来实现 "协程 Coroutine" 吗?

答案肯定是否定. 

它还可以用于面向切片编程, 例如实现一些前后端网络框架里的"拦截器".

它还可以用于实现类似于 java 里的动态代理.

它还可以用于实现动态插件化.

它还可以用于实现不停机更新程序修复 bug,  这是个一般做云服务做得比较深入的同学才会触碰的话题....😜


### Boost.Context 带来的 DSL 语言

多线程编程中, 代码充斥着大量的 "Callback Hell", 因此出现 "如何以同步代码的形式写异步代码" 的研究课题.

目前在一些语言里已内置此功能:

如 `JavaScript` `Dart` 里的 `Future` 类和为之特意设计的语法糖 `aysnc` `await`.

如 `kotlin-coroutines` 里面的 `suspend` 方法.

如 Golang lua 里面的内置的协程语法糖.

上面种种语言的实现方式其实都是设计一些语法糖然后最终通过编译技术生成对应的 `Future` 对象来实现的. 所以下面借助

[Swift_Boost_Context](https://github.com/Guang1234567/Swift_Boost_Context)

来实现一个"基于单线程的简单的 Futrue"


```swift

import Foundation
import Swift_Boost_Context

public enum CoFutureError: Error {
    case canceled
}

public class CoFuture<R>: CustomDebugStringConvertible, CustomStringConvertible {

    let _name: String

    let _task: () throws -> R

    var _result: Result<R, Error>?

    var _bctx: BoostContext!

    deinit {
        print("\(self) : deinit")
    }

    public init(_ name: String, _ task: @escaping () throws -> R) {
        self._name = name
        self._task = task
        self._result = nil

        self._bctx = makeBoostContext { [unowned self] (fromCtx: BoostContext, data: Void) -> Void in
            let result: Result<R, Error> = Result { [unowned self] in
                try self._task()
            }

            let _: BoostTransfer<Void> = fromCtx.jump(data: result)
        }
    }

    @discardableResult
    public func await() throws -> R {
        let btf: BoostTransfer<Result<R, Error>> = self._bctx.jump(data: ())
        return try btf.data.get()
    }

    public func cancel() -> Void {
        if self._result == nil {
            self._result = .failure(CoFutureError.canceled)
        }
    }

    public var isCanceled: Bool {
        if case .failure(let error as CoFutureError)? = self._result {
            return error == .canceled
        }
        return false
    }

    public var debugDescription: String {
        return "CoFuture(_name: \(_name))"
    }

    public var description: String {
        return "CoFuture(_name: \(_name))"
    }
}

```

用法:

``` swift
CoFuture("01") {
    
    let r11 = CoFuture("01_01") {
       
        return 11
    }
    .await()
    
    let r12 = CoFuture("01_02") {
       
        return 12
    }
    .await()
    
    return r11 + r12
}

```

其实上面我们使用 `Swift` 创建了一套关于 `Future` 的 DSL 语言.

## 技术细节二:  实现 Coroutine

结合 Swift `GCD` 和 `BoostContext` 打造一个 API 类似于 `Kotlin-Coroutines` 的 `Coroutines` 类.

它像 `Kotlin-Coroutines` 一样, 支持`多线程`环境下的协程, 而不是单线程!

另外重复 3 遍, Coroutine 是 `非阻塞 (Non-Blocking)`, `非阻塞`, `非阻塞` 的.

```swift
import Foundation
import Swift_Boost_Context
import SwiftAtomics
import RxSwift
import RxCocoa
import RxBlocking

public enum CoroutineState: Int {
    case INITED = 0
    case STARTED = 1
    case RESTARTED = 2
    case YIELDED = 3
    case EXITED = 4
}

public typealias CoroutineScopeFn<T> = (Coroutine) throws -> T

public typealias CoroutineResumer = () -> Void

public class CoJob {

    var _isCanceled: AtomicBool
    let _co: Coroutine

    public var onStateChanged: Observable<CoroutineState> {
        self._co.onStateChanged
    }

    init(_ co: Coroutine) {
        self._isCanceled = AtomicBool()
        self._isCanceled.initialize(false)
        self._co = co
    }

    @discardableResult
    public func cancel() -> Bool {
        // can not cancel Coroutine
        _isCanceled.CAS(current: false, future: true)
    }

    public func join() throws -> Void {
        try _co.onStateChanged.ignoreElements().toBlocking().first()
    }
}

public protocol Coroutine {

    var currentState: CoroutineState { get }

    var onStateChanged: Observable<CoroutineState> { get }

    func yield() throws -> Void

    func yieldUntil(cond: () throws -> Bool) throws -> Void

    func yieldUntil(_ beforeYield: (@escaping CoroutineResumer) -> Void) throws -> Void

    func delay(_ timeInterval: DispatchTimeInterval) throws -> Void

}

enum CoroutineTransfer<T> {
    case YIELD
    case YIELD_UNTIL(Completable)
    case DELAY(DispatchTimeInterval)
    case EXIT(Result<T, Error>)
}

class CoroutineImpl<T>: Coroutine, CustomDebugStringConvertible, CustomStringConvertible {

    let _name: String

    var _originCtx: BoostContext!

    var _yieldCtx: BoostContext?

    let _dispatchQueue: DispatchQueue

    let _task: CoroutineScopeFn<T>

    var _currentState: AtomicInt

    let _disposeBag: DisposeBag = DisposeBag()

    let _onStateChanged: AsyncSubject<CoroutineState>

    var currentState: CoroutineState {
        CoroutineState(rawValue: _currentState.load()) ?? .EXITED
    }

    var onStateChanged: Observable<CoroutineState> {
        return _onStateChanged.asObserver()
    }

    deinit {
        self._originCtx = nil
        self._yieldCtx = nil
        //print("CoroutineImpl deinit : _name = \(self._name)")
    }

    init(
            _ name: String,
            _ dispatchQueue: DispatchQueue,
            _ task: @escaping CoroutineScopeFn<T>
    ) {
        self._name = name
        self._onStateChanged = AsyncSubject()
        self._yieldCtx = nil
        self._dispatchQueue = dispatchQueue
        self._task = task
        self._currentState = AtomicInt()
        self._currentState.initialize(CoroutineState.INITED.rawValue)

        // issue: memory leak!
        //self.originCtx = makeBoostContext(self.coScopeFn)

        self._originCtx = makeBoostContext { [unowned self] (fromCtx: BoostContext, data: Void) -> Void in
            //print("\(self)  coScopeFn  :  \(fromCtx)  ---->  \(_bctx!)")
            self._currentState.CAS(current: CoroutineState.INITED.rawValue, future: CoroutineState.STARTED.rawValue)
            self.triggerStateChangedEvent(.STARTED)

            self._yieldCtx = fromCtx
            let result: Result<T, Error> = Result { [unowned self] in
                try self._task(self)
            }

            //print("\(self)  coScopeFn  :  \(self._fromCtx ?? fromCtx)  <----  ")
            let _: BoostTransfer<Void> = (self._yieldCtx ?? fromCtx).jump(data: CoroutineTransfer.EXIT(result))
            //print("Never jump back to here !!!")
        }
    }

    func triggerStateChangedEvent(_ state: CoroutineState) {
        self._onStateChanged.on(.next(state))
        if state == CoroutineState.EXITED {
            self._onStateChanged.on(.completed)
        }
    }

    func start() -> Void {
        let bctx: BoostContext = self._originCtx
        self._dispatchQueue.async(execute: self.makeResumer(bctx))
    }

    func resume(_ bctx: BoostContext, ctf: CoroutineTransfer<T>) -> Void {
        switch ctf {
            case .YIELD:
                //print("\(self)  --  YIELD")
                triggerStateChangedEvent(.YIELDED)
                //self._dispatchQueue.asyncAfter(deadline: .now() + .milliseconds(5), execute: self.makeResumer(bctx))
                self._dispatchQueue.async(execute: self.makeResumer(bctx))
                //print("\(self)  --  YIELD  -- finish")
            case .YIELD_UNTIL(let onJumpBack):
                //print("\(self)  --  YIELD_UNTIL")
                triggerStateChangedEvent(.YIELDED)
                onJumpBack.subscribe(onCompleted: {
                              //print("\(self)  --  YIELD_UNTIL2")
                              self._dispatchQueue.async(execute: self.makeResumer(bctx))
                          })
                          .disposed(by: self._disposeBag)
            case .DELAY(let timeInterval):
                //print("\(self)  --  DELAY  --  \(timeInterval)")
                triggerStateChangedEvent(.YIELDED)
                self._dispatchQueue.asyncAfter(deadline: .now() + timeInterval, execute: self.makeResumer(bctx))
                //print("\(self)  --  DELAY -- finish")
            case .EXIT(let result):
                //print("\(self)  --  EXITED  --  \(result)")
                self._currentState.store(CoroutineState.EXITED.rawValue)
                triggerStateChangedEvent(.EXITED)

        }
    }

    func makeResumer(_ bctx: BoostContext) -> CoroutineResumer {
        return { [unowned self] in
            let btf: BoostTransfer<CoroutineTransfer<T>> = bctx.jump(data: ())
            let coTransfer: CoroutineTransfer<T> = btf.data
            return self.resume(btf.fromContext, ctf: coTransfer)
        }
    }

    func yield() throws -> Void {
        return try self._yield(CoroutineTransfer.YIELD)
    }

    func yieldUntil(cond: () throws -> Bool) throws -> Void {
        while !(try cond()) {
            try self.yield()
        }
    }

    func yieldUntil(_ beforeYield: (@escaping CoroutineResumer) -> Void) throws -> Void {
        let resumeNotifier: AsyncSubject<Never> = AsyncSubject()
        beforeYield({ resumeNotifier.on(.completed) })
        try self._yield(CoroutineTransfer.YIELD_UNTIL(resumeNotifier.asCompletable()))
    }

    func delay(_ timeInterval: DispatchTimeInterval) throws -> Void {
        try self._yield(CoroutineTransfer.DELAY(timeInterval))
    }

    func _yield(_ ctf: CoroutineTransfer<T>) throws -> Void {
        // not in current coroutine scope
        // equals `func isInsideCoroutine() -> Bool`
        // ---------------
        guard let yieldCtx = self._yieldCtx else {
            throw CoroutineError.calledOutsideCoroutine(reason: "Call `yield()` outside Coroutine")
        }

        // jump back
        // ---------------
        _currentState.store(CoroutineState.YIELDED.rawValue)
        //print("\(self)  _yield  :  \(fromCtx)  <----  \(Thread.current)")
        let btf: BoostTransfer<Void> = yieldCtx.jump(data: ctf)
        // update `self._fromCtx` when restart
        self._yieldCtx = btf.fromContext
        _currentState.store(CoroutineState.RESTARTED.rawValue)
        triggerStateChangedEvent(.RESTARTED)
        //print("\(self)  _yield  :  \(btf.fromContext)  ---->  \(Thread.current)")
    }

    func isInsideCoroutine() -> Bool {
        return self._yieldCtx != nil
    }

    var debugDescription: String {
        return "CoroutineImpl(_name: \(_name))"
    }
    var description: String {
        return "CoroutineImpl(_name: \(_name))"
    }
}


public class CoLauncher {
    public static func launch<T>(
            name: String = "",
            dispatchQueue: DispatchQueue,
            _ task: @escaping CoroutineScopeFn<T>
    ) -> CoJob {
        let co: CoroutineImpl = CoroutineImpl<T>(name, dispatchQueue, task)
        co.start()
        return CoJob(co)
    }
}


extension CoroutineImpl: ReactiveCompatible {

}


extension Reactive where Base: CoroutineImpl<Any> {

}
```


用法:

```swift
func example_01() throws {
    // Example-01
    // ===================
    print("Example-01 =============================")

    //let queue = DispatchQueue(label: "TestCoroutine")
    let queue = DispatchQueue.global()

    let coJob1 = CoLauncher.launch(name: "co1", dispatchQueue: queue) { (co: Coroutine) throws -> String in
        defer {
            print("co 01 - end \(Thread.current)")
        }
        print("co 01 - start \(Thread.current)")
        try co.yield()
        return "co1 's result"
    }

    let coJob2 = CoLauncher.launch(dispatchQueue: queue) { (co: Coroutine) throws -> String in
        defer {
            print("co 02 - end \(Thread.current)")
        }
        print("co 02 - start \(Thread.current)")
        try co.yield()
        throw TestError.SomeError(reason: "Occupy some error in co2")
        return "co2 's result"
    }

    let coJob3 = CoLauncher.launch(dispatchQueue: queue) { (co: Coroutine) throws -> String in
        defer {
            print("co 03 - end \(Thread.current)")
        }
        print("co 03 - start \(Thread.current)")
        try co.yield()
        return "co3 's result"
    }

    try coJob1.join()
    try coJob2.join()
    try coJob3.join()

    print("Example-01 =============  end  ===============")
}
```

控制台输出:

```ruby
Example-01 =============================
co 01 - start <NSThread: 0x7f9e3c004600>{number = 3, name = (null)}
co 02 - start <NSThread: 0x7f9e3a4060f0>{number = 2, name = (null)}
co 03 - start <NSThread: 0x7f9e3c104120>{number = 4, name = (null)}
co 01 - end <NSThread: 0x7f9e3a4060f0>{number = 2, name = (null)}
co 03 - end <NSThread: 0x7f9e3c004600>{number = 3, name = (null)}
co 02 - end <NSThread: 0x7f9e3c104120>{number = 4, name = (null)}
Example-01 =============  end  ===============
```

正如上面的控制台输出所示:

创建了 3 个协程, 每个协程运行到中途 "挂起", 让出被 cpu 执行的权利. 故会发现每个协程的"前半部分代码"和"后半部分代码"执行时所处的线程不一样!!!


## 技术细节三:  实现基于 Coroutine 的 Future

具体实现:

[CoFuture.swift](https://github.com/Guang1234567/Swift_Coroutine/blob/master/Sources/Swift_Coroutine/concurrent/CoFuture.swift)

用法:

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

值得注意的是, 可以指定 `CoFuture` 内部所使用的线程,
在这里直接依赖于 `Swift GCD` 提供的线程池类 `DispatchQueue` , 等价于 Java 生态的 `ThreadPoolExecutor`.

可以为每个 `CoFuture` 指定 `DispatchQueue` 意味着每个子任务可以运行在不同的线程, 然后以同步代码的形式组织起一些异步子任务, 如上面例子 `workflow` 方法. 

这有点类似于 `RxJava` `RxSwift` 里的 `observeOn` 操作符, 简单快捷地为每个子任务指定所使用的线程.

```java
// java 代码
// ==================

Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> observableEmitter) throws Exception {
                // ...
            }
        });
        observable.subscribeOn(Schedulers.newThread())
                .observeOn(Schedulers.io())
                .map(new Function<Integer, Integer>() {
                    // ...
                })
                .observeOn(AndroidSchedulers.mainThread())
                .doOnSubscribe(new Consumer<Disposable>() {
                    // ...
                })
                .subscribeOn(Schedulers.single())
                .subscribe(new Consumer<Integer>() {
                    // ...
                });
```

用 `Kotlin-Coroutine` 的 `suspend` 方法和 `withContext` 也能完成同样的事情.

```kotlin
// kotlin 代码
// ===================

suspend func loadSpeaker() : Result<Speaker> {
    withContext(Dispatchers.IO) {
        // running on  "IO"  thread
    }
}

suspend func nextTalk() : Result<Talk> {
    withContext(Dispatchers.Default) {
            // running on  "New"  thread
    }
}

suspend func getConference() : Result<Conference> {
     withContext(Dispatchers.IO) {
            // running on  "IO"  thread
        }
}

suspend func getCity() : Result<City> {
     withContext(Dispatchers.Main) {
            // running on  "Main"  thread
        }
}
```

## 技术细节四:  实现基于 Coroutine 的 Channel (通道)

大家最熟悉的“生产者-消费者”事件驱动模型，一个线程负责生产产品并将它们加入队列，另一个负责从队列中取出产品并使用它。

实际上多线程在此的应用还是显得有点“重量级”，由于缺乏 yield 语义，线程之间不得不使用同步机制来避免产生全局资源的竟态，这就不可避免产生了休眠、调度、切换上下文一类的系统开销，而且线程调度还会产生时序上的不确定性。而对于协程来说，“挂起”的概念只不过是转让代码执行权并调用另外的协程，待到转让的协程告一段落后重新得到调用并从挂起点“唤醒”，这种协程间的调用是逻辑上可控的，时序上确定的，可谓一切尽在掌握中.

具体实现:

[CoChannel.swift](https://github.com/Guang1234567/Swift_Coroutine/blob/master/Sources/Swift_Coroutine/concurrent/CoChannel.swift)

用法:

```swift
func example_05() throws {
    // Example-05
    // ===================
    print("Example-05 =============================")

    let producerQueue = DispatchQueue(label: "producerQueue", attributes: .concurrent)
    let consumerQueue = DispatchQueue(label: "consumerQueue", attributes: .concurrent)
    let closeQueue = DispatchQueue(label: "closeQueue", attributes: .concurrent)
    let channel = CoChannel<Int>(capacity: 7)

    let coClose = CoLauncher.launch(name: "coClose", dispatchQueue: closeQueue) { (co: Coroutine) throws -> Void in
        print("coClose before  --  delay")
        try co.delay(.milliseconds(10))
        //try co.yield()
        print("coClose after  --  delay")
        channel.close()
        print("coClose  --  end")
    }

    let coConsumer = CoLauncher.launch(name: "coConsumer", dispatchQueue: consumerQueue) { (co: Coroutine) throws -> Void in
        var time: Int = 1
        for item in try channel.receive(co) {
            print("consumed : \(item)  --  \(time)  --  \(Thread.current)")
            time += 1
        }
    }

    let coProducer01 = CoLauncher.launch(name: "coProducer01", dispatchQueue: producerQueue) { (co: Coroutine) throws -> Void in
        for time in (1...32).reversed() {
            //print("coProducer01  --  before produce : \(time)")
            try channel.send(co, time)
            print("coProducer01  --  after produce : \(time)")
            try co.delay(.milliseconds(1))
        }
        print("coProducer01  --  end")
    }

    /*let coProducer02 = CoLauncher.launch(name: "coProducer02", dispatchQueue: producerQueue) { (co: Coroutine) throws -> Void in
        for time in (33...50).reversed() {
            //print("coProducer02  --  before produce : \(time)")
            try channel.send(co, time)
            print("coProducer02  --  after produce : \(time)")
        }
        print("coProducer02  --  end")
    }*/

    try coClose.join()
    try coConsumer.join()
    try coProducer01.join()
    //try coProducer02.join()

    print("channel = \(channel)")
}
```

此 `CoChannel` 的实现支持 `多生产者-单消费者`, 把上面例子的注释打开运行一下即可演示.


## 技术细节五:  实现基于 Coroutine 的 Semaphore (信号量)

具体实现:

[CoSemaphore.swift](https://github.com/Guang1234567/Swift_Coroutine/blob/master/Sources/Swift_Coroutine/concurrent/CoSemaphore.swift)

信号量也能有`非阻塞`的版本...😂😭😳

用法:

```swift
func example_02() throws {
    // Example-02
    // ===================
    print("Example-02 =============================")

    let producerQueue = DispatchQueue(label: "producerQueue", attributes: .concurrent)
    let consumerQueue = DispatchQueue(label: "consumerQueue", attributes: .concurrent)
    let semFull = CoSemaphore(value: 8, "full")
    let semEmpty = CoSemaphore(value: 0, "empty")
    let semMutex = DispatchSemaphore(value: 1)
    var buffer: [Int] = []

    let coConsumer = CoLauncher.launch(dispatchQueue: consumerQueue) { (co: Coroutine) throws -> Void in
        for time in (1...32) {
            try semEmpty.wait(co)
            semMutex.wait()
            if buffer.isEmpty {
                fatalError()
            }
            let consumedItem = buffer.removeFirst()
            print("consume : \(consumedItem)  -- at \(time)   \(Thread.current)")
            semMutex.signal()
            semFull.signal()
        }
    }

    let coProducer = CoLauncher.launch(dispatchQueue: producerQueue) { (co: Coroutine) throws -> Void in
        for time in (1...32).reversed() {
            try semFull.wait(co)
            semMutex.wait()
            buffer.append(time)
            print("produced : \(time)   \(Thread.current)")
            semMutex.signal()
            semEmpty.signal()
        }
    }

    try coConsumer.join()
    try coProducer.join()

    print("finally, buffer = \(buffer)")
    print("semFull = \(semFull)")
    print("semEmpty = \(semEmpty)")
}
```

是不是有点像上面 `CoChannel` 的例子, 其实 `CoChannel` 的实现离不开 `CoSemaphore`. 😜


## 总结

`Swift`可以用来构建 Web App, 目前也有类似于 [vapor](https://github.com/vapor/vapor) 的 Web 框架 (跟 spring 全家桶一样全😂).

[vapor](https://github.com/vapor/vapor) 是基于 `NIO` 实现的一套框架, 接下来计划可能会给它加入 `Coroutine` 特性, 实现一个类似于 [kotlin/ktor](https://ktor.kotlincn.net/) 的 Web 框架.

当然 `Coroutine` 也能用于 Mobile App, Desktop App, 

作为全干工程师一些核心的轮子还是要自己重新实现一遍, 实现的过程就是精通原理和运用的过程. 

哪怕出现一个新的协程库, 估计把文档看一遍就能把它运用得出神入化.

正所谓 “一法通，万法通”.
