---
title: Swift 遇上 Android
abbrlink: 8d4de98e
tags:
  - Swift
  - Android
  - CrossPlatform
copyright: true
sticky: false
hide: false
cover: /2020/08/10/swift-with-android/Swift_With_Android.png
top_img: /2020/08/10/swift-with-android/Swift_With_Android.png
categories:
  - [Swift]
  - [Android]
keywords: 'Coroutine,Swift,CrossPlatform'
description: 一个用 Swift 和汇编语言编写的协程库
katex: false
date: 2020-08-10 18:13:47
---


## (1) 简介

`Rust` 提供 `Android` 交叉编译工具链, 支持 `arm64-v8a`, `armeabi-v7a`,  `x86`, `x86_64`.

同样, `Swift` 也提供 `Android` 交叉编译工具链, 也支持 `arm64-v8a`, `armeabi-v7a`,  `x86`, `x86_64`.

但 `Swift` 的 `Android` 交叉编译工具链的二进制安装包在 `Apple` 旗下相关的网站 [swift.org](https://swift.org/download/) 是无法找到的, 具体原因想想都知道啦, 😝.

然后, 🇺🇸旧金山硅谷一家叫 `Readdle` 的公司, 帮我们编译了一个, 相关资料文档和编译工具链安装包和用法在 [Readdle Blog 的 Swift for Android: Our Experience and Tools](https://blog.readdle.com/why-we-use-swift-for-android-db449feeacaf) 这篇官方博客有说.

但 `Readdle` 公司提供的 `Android交叉编译工具链` 是基于  swift-v5.0.x 的源码编译的.
另外, 截止于 2020年05月20日, Swift 的编译器已经是  swift-v5.2.4, 请看下图:


![](1.png)

这个图信息量其实是有点大的:

- `绿框` 的编译链工具表示所支持的一些 Linux 发行版本, 只要国内的 `XXX 云` 支持上面的环境, 基本上都能部署 Swift 编写的 Web App.

- `红框` 的就不多说了.

- 怎么不支持微软 windows 呢? 计划今年的 Swift 5.3.x 才会开始官方支持(其实官方的 swift repo 已经有相关 commit),  也就是说 github 能找到非官方的 O(∩_∩)O.

----

那么最重点的来了, 最新的 `swift-android-toolchain-v5.2.x` 交叉编译工具链在哪里?

在 [这里](https://github.com/Guang1234567/Swift_Android_Toolchains)

![](2.png)

下载 `绿框` 所示的 `swift-android-toolchain-v5.2.3` 就行了, 这些都是我基于 `NDK-20` 编译的,  完整支持 `android-api-24`


## (2) 如何进行编译?

### (2.1)  只使用命令行

例如, 要编译 `arm64-v8a` 架构的 `*.so`, 在 terminal 输入

```bash
export SWIFT_ANDROID_ARCH=aarch64

export SWIFT_ANDROID_HOME=$HOME/dev_kit/sdk/swift_source/swift-android-5.2.3-release

${HOME}/dev_kit/sdk/swift_source/swift-android-5.2.3-release/build-tools/1.9.6-swift5/swift-build --configuration debug -Xswiftc -DDEBUG -Xswiftc -g
```

**编译的 output :**

```ruby
// ....

[102/102] Linking libnative-activity.so

######################## Swift-Android-Toolchain ########################

${HOME}/dev_kit/sdk/swift_source/swift-android-5.2.3-release/toolchain/usr/bin/swift-build --destination=<(${HOME}/dev_kit/sdk/swift_source/swift-android-5.2.3-release/build-tools/1.9.6-swift5/src/bash/generate-destination-json.sh) -Xcc -I.build/jniLibs/include -Xswiftc -I.build/jniLibs/include -Xswiftc -L.build/jniLibs/arm64-v8a -Xlinker -L.build/jniLibs/arm64-v8a -Xcc -D__linux__ -Xcc -D__ANDROID__ -Xcc -D__ANDROID_API_N__=24 -Xcc -D__ANDROID_API_M__=23 -Xcc -D__ANDROID_API__=24 -Xcc -DDEPLOYMENT_TARGET_ANDROID -Xcc -DDEPLOYMENT_TARGET_LINUX -Xcc -DDEPLOYMENT_RUNTIME_SWIFT -Xcc -funwind-tables --configuration debug -Xswiftc -DDEBUG -Xswiftc -g

Compile End

#########################################################################
```

### (2.2) 使用 Android studio 3.6

其实有个 gradle 插件已经封装了上面的命令上, 把它集成到 `build.gradle` 即可, 如:

```gradle
// File: build.gradle
// ================

buildscript {
    repositories {
       google()
       jcenter()
       maven { url "https://dl.bintray.com/readdle/maven" }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.2'
        classpath 'com.readdle.android.swift:gradle:1.3.3'
    }
}
```

然后呢? 当然是点击 studio 那个 绿色的 "△" 按钮编译运行.  That's all.


最后, 截个图作为证据吧, 省得还以为是骗人的....


![](3.png)


![](4.png)

留意一下上面截图中绿色箭头指向的 `import SkiaSwift` 和 `import SGLOpenGL`, 下面的章节会有介绍.

## (3) 生态

写个 app 不可能轮子全部造吧 (⊙﹏⊙)b).

其实你在 github 上搜索到的用 `pure swift` 为 `iOS` 写的库基本上都能用在 Android 上, 如:

- 数据库必须要有!   github 上搜索 `sqlite swift` 一大堆.

- 网络通信怎么办? 直接在 Android 上使用 `UrlSession` 这个 `apple 的 api`, 是不是感觉有点虚幻?

```swift
let sessionConfigure = URLSessionConfiguration.default 
sessionConfigure.httpAdditionalHeaders = ["Content-Type": "application/json"]
sessionConfigure.timeoutIntervalForRequest = 30
sessionConfigure.requestCachePolicy = .reloadIgnoringLocalCacheData
let session = URLSession(configuration: sessionConfigure)

 guard let url =  URL(string: "http://rap.taobao.org/mockjsdata/13552/provincelist") else {
        return
    }
 let urlRequest = URLRequest(url: url)
 let dataTask = session.dataTask(with: urlRequest) { (data, response, error) in
     guard let resultData = data else {
         return
 }
  do {
     let jsonObject = try JSONSerialization.jsonObject(with: resultData, options: .mutableLeaves)
         print(jsonObject)
   } catch {
         print("解析错误！")
   }
 }
```

用惯 `Okhttp` 的 Android 同学表示上面要写这么多行差评!   其实 github 有很多它的 `swift` 版本,都是基于 `URLSession` or  `curl` 二次封装的,这里就不举例了.

Apple 阵营的程序员表示:  感觉果粉的世界要被玷污了, 被隔壁的老王(Android).

- json 和 xml ? 这个不用说吧.

-  kotlin 的协程, Go 的协程, Elixir 的"进程(协程)",  其中 Go 和 Elixir 写 Web app 的同学应该不陌生,  但我要说的是 [Swift Coroutine](https://github.com/belozierov/SwiftCoroutine), 下面来跟 `Kotlin Coroutine`的协程对比一下:

```swift
// swift coroutine example
// ====================

//execute coroutine on the main thread
DispatchQueue.main.startCoroutine {
	// ......
}
```

**vs**

```kotlin
// kotlin coroutine example
// ====================

//execute coroutine on the main thread
GlobalScope.launch(Dispatchers.Main) { 
	// ......
}
```

Go 和 Elixir 的版本就不在这里列举了.

  在这里我想告诉服务器的同学:  swift kotlin 写服务器也可以有协程,  处理`高并发`的线程模型不再是独一无二了, O(∩_∩)O哈哈哈~.

- 线程方面:  Android 的 EventLooper 和 java 的ThreadExecutor  locker Semaphore? 在 swift 的生态都有对应物, 也是很方便实用的!   

下面举个 `工作线程运行耗时操作并把结果返回到主线程刷新 UI`的例子:

```java
//  Android
// ===========
Executors.newFixedThreadPool(7).execute(

         Log.i("log_tag", "开始执行异步任务")
         Thread.sleep(100)
         Log.i("log_tag", "异步任务执行完毕")
         
       new Handler(Looper.getMainLooper()).post ({
                 Log.i("log_tag", "回到主线程刷新UI")
       });
);
```

```swift
// iOS
// =================

DispatchQueue.global().async {

         print("开始执行异步任务")
         Thread.sleep(forTimeInterval: 100)
         print("异步任务执行完毕")
         
         DispatchQueue.main.async {
             print("回到主线程刷新UI")
         }
         }
     }
```

能想象到上面这段 ios 代码竟能跑在 Android 上吗?  竟然变成了真实.

再来个能跑在 Android 上的 swift 多线程代码:

```swift
// 使用队列实现 "Producer-Consumer" 生产者消费者模式 o(￣▽￣)d
// ===================================================

/*
Dispatch Barrier

GCD’s barrier API ensures that the submitted block is the only item executed on the specified queue for that particular time. 
This means that all items submitted to the queue prior to the dispatch barrier must complete before the block will execute.
*/

let dispatchQueue = DispatchQueue(
	label: "com.prit.TestGCD.DispatchQueue", 
	attributes: DispatchQueueAttributes.concurrent)

//Producer:
func add(num: Int) {
    dispatchQueue.async(flags: .barrier) {
        print("Add \(num)")
        self.items.append(num)
    }
}
    
//Consumer:
func remove() {
    dispatchQueue.async(flags: .barrier) {
        
        guard !self.items.isEmpty else {
            return
        }
        let num = self.items.removeLast()
        print("Remove \(num)")
    }
}
```

再再来个能跑在 Android 上的 swift 多线程代码:

```swift
// 信号量  o(￣▽￣)d
// ===================================================

let semaphore = DispatchSemaphore(value: 1)
    
//Producer:
func add(num: Int) {
    semaphore.wait(timeout: DispatchTime.distantFuture)
    print("Add \(num)")
    items.append(num)
    semaphore.signal()
}
    
//Consumer:
func remove() {
    semaphore.wait(timeout: DispatchTime.distantFuture)
    defer {
        semaphore.signal()
    }
    guard !items.isEmpty else {
        return
    }
    let num = items.removeLast()
    print("Remove \(num)")
}
```

- WebView 怎么办?  只能像 flutter 框架那样通过 methodchannel 来调用原生的 webview 来显示网页.  由于有 `jni` (充当 methonchannel), 那么 swift 可以通过 `jni` 去调动原生的 webview.

- `rxJava  rxAndroid`  **vs**  `rxSwift`

- 其他还没想到的 (待补充)


### (3.1) 跨平台 UI 框架

看完上面的章节, 终于来到最重要的环节了, UI 怎么办?  

我想要在 iOS, Android, Linux, Windows, 甚至是 Web 上共用同一套 UI.

那目前有没有用纯 swift 写的 UI 框架, 并且通过交叉编译链工具使之能在多个平台上正常运行? 

答案: 是没有.

但 Apple 有套叫 `SwiftUI` 的响应式框架, 但只在自家的产品上的"跨平台" (好像包括 iOS macOS tvOS ...), 应该肯定不开放源码. 

如果开源的话, 早就没 flutter 什么事情了. 

因为我有交叉编译工具链 (⊙﹏⊙)b,  只需把底层的2d渲染引擎换成一个类似于 google skia 的跨平台2d 渲染引擎, 就能在 Android 上运行 iOS 风格的 APP.

而且还是纯原生的, 不依赖虚拟机的 (swift 是不依赖于高级 VM 的),
从而在 UI 绘制方面跟 iOS App 一样的快! 

> 题外话: 
> <br/>
> 最近 flutter 很流行呀, 这是事实!  
> 如果用过,会发现 [Cupertino(iOS风格)的 widget](https://flutterchina.club/widgets/cupertino) 不是很全(很少), 或多或少有点 bug, Google 也没把尽力完善它.  为啥不完善呢?
>  <br/>
>  其实 google 也知道可以借助 swift 生态搭建一些类 Linux OS, 为啥不用呢?
>  还想在 Fusia 用 Rust ....

---

是时候用纯 swift 打造一套类似于 `SwiftUI`, `Flutter`, `React` (不是 `ReactNative`) 的 UI 框架来解决这个问题了, 而不是抱怨这个问题.

得益于互联网上有 `skia` 这样现成的 2d 渲染引擎, 并且 `Flutter`, `React` 的上层建筑(`Widget`) 代码都是开源的, 其实只要用 swift 把 `widget` 这一层重新"翻译", 并把它移植在 `skia` 上这个花盆上就行了 (flutter 也是植根于 `skia` 上的, 有参照物了, 哈哈哈, 不如命名为 `Flutter-Swift-CrossPlatform`, 以区别 google 的 `Flutter-Dart-CrossPlatform`).

其实 github 上已经有一些开源项目:

+ 可运行于 Linux (Android 也算是个 Linux 的发行版本)上:
	- [Cosmo/OpenSwiftUI](https://github.com/Cosmo/OpenSwiftUI) — OpenSwiftUI is an OpenSource implementation of Apple's SwiftUI DSL.
	- [Cosmo/awesome-embedded-swift](https://github.com/Cosmo/awesome-embedded-swift) — SwiftUIEmbedded is an implementation of SwiftUI (based on OpenSwiftUI) for embedded and Linux devices.

+ 可运行于浏览器的
 	- [SwiftWebUI/SwiftWebUI](https://github.com/SwiftWebUI/SwiftWebUI)

看来大家都喜欢 `Open-SwiftUI-CrossPlatform` 多过于 `Flutter-Swift-CrossPlatform`, 哈哈哈.

所以目前的计划是尝试通过将  [Cosmo/OpenSwiftUI](https://github.com/Cosmo/OpenSwiftUI) 桥接 `skia` 移植到 `Android` 上, 是不是听起来会秃顶的样子...


### (3.2) Skia And Swift

要想完成上面的工作, 需要 [Skia_Swift_CrossPlatform](https://github.com/Guang1234567/Skia_Swift_CrossPlatform)

是基于 [SkiaSharp](https://github.com/mono/SkiaSharp) fork 过来改的.

> 简介:
> <br/>
> SkiaSharp is a cross-platform 2D graphics API for .NET platforms based on Google's Skia Graphics Library. It provides a comprehensive 2D API that can be used across mobile, server and desktop models to render images.
> <br/>
> <br/>
>  微软也用它来做手机方面的跨平台.
> <br/>
> <br/>
> [API 文档](https://docs.microsoft.com/zh-cn/dotnet/api/skiasharp.grcontext?view=skiasharp-1.68.1)

除此之外,  还需要使用 [Swift_Opengl](https://github.com/Guang1234567/Swift_OpenGL/tree/gles32_egl15_android/Sources/SGLOpenGL) 和 [Swift_EGL](https://github.com/Guang1234567/Swift_OpenGL/tree/gles32_egl15_android/Sources/SGLEGL)  创建 GPU 渲染环境 (硬件加速).

当然你可以在 JVM 的世界通过 [GLSurfaceView](https://developer.android.com/guide/topics/graphics/opengl?hl=zh-cn)  和 [Swift_Android_NativeWindow](https://github.com/Guang1234567/Swift_Android_NativeWindow) 来创建 GPU 渲染环境.

一旦 GPU 渲染环境搭建好了,  [Skia_Swift_CrossPlatform](https://github.com/Guang1234567/Skia_Swift_CrossPlatform) 就能在它上面运行,以获取硬件加速.

当然你可以选择 CPU 来进行软件绘制的 ⊙﹏⊙|||.


[Skia_Swift_CrossPlatform](https://github.com/Guang1234567/Skia_Swift_CrossPlatform) +  [Swift_Opengl](https://github.com/Guang1234567/Swift_OpenGL/tree/gles32_egl15_android/Sources/SGLOpenGL) +  [Swift_EGL](https://github.com/Guang1234567/Swift_OpenGL/tree/gles32_egl15_android/Sources/SGLEGL) 的 Android App Example 在[这里](https://github.com/Guang1234567/Swift_Android_Glue/tree/master/Examples/Android/native-activity/app/src/main/swift).

来自 huawei play 真机的截图: 

![](5.png)

```swift
// File: native-activity.swift
// 上面截图的对应的代码, 主要是画了条"贝塞尔曲线"
// ==================

class SkEglRendererImpl: SkEglRenderer {

    func onSurfaceCreated(_ surface: Surface) {}

    func onSurfaceChanged(_ width: EGLint, _ height: EGLint) {}

    func onDrawFrame(_ canvas: Canvas) {
        canvas.clear(Color(r: 0, g: 0xB9, b: 0x5B))
        drawBySkiaEngine(canvas)
    }

    func drawBySkiaEngine(_ canvas: Canvas) {
        let fill = Paint()
        fill.color = Color(r: 0, g: 0, b: 0xFF)
        canvas.drawPaint(fill)

        fill.color = Color(r: 0, g: 0xFF, b: 0xFF)
        let rect = Rect(100.0, 100.0, 540.0, 380.0)
        canvas.drawRect(rect, fill)

        let stroke = Paint()
        stroke.color = Color(r: 0xFF, g: 0, b: 0)
        stroke.antialias = true
        stroke.stroke = true
        stroke.strokeWidth = 5.0

        let path = Path()
        path
                .moveTo(50.0, 50.0)
                .lineTo(590.0, 50.0)
                .cubicTo(-490.0, 50.0, 1130.0, 430.0, 50.0, 430.0)
                .lineTo(590.0, 430.0)
        canvas.drawPath(path, stroke)

        fill.color = Color(r: 0, g: 0xFF, b: 0, a: 0x80)
        let rect2 = Rect(120.0, 120.0, 520.0, 360.0)
        canvas.drawOval(rect2, fill)
    }
}
```

要想看懂上面提及所有内容和相关依赖库和例子, 需要:

+ 学习: 
	- Swift 语言和生态系统和工具(如: SPM)
	- 少量 C 和 C++ 知识, 最起码得看得懂指针 
	- Swift 如何调用 C
	- Swift 的并发库: GCD
	- OpenGL
	- EGL			- 创建 GPU Surface 用的, UI 渲染在上面.
	- Skia 的简单用法(其实就是 Android 的 Canvas)`
	- 由于需要预编译一些库, 需要一点 cmake 和 makefile 的知识. (虽然都预编译好打包好给大家)
	-  Android NDK , 用来编译一些 C 库,  ndk cmake 和 android.mk 总得了解下吧
	- 一点 bash 脚本的知识
	- Android app jvm 层的知识
	- 其他(待补充)

+ 设备和系统:
	- ubuntu  or linuxMint (以前用它编译 android 系统源码, 以上内容所提及的所有代码均没有在这两系统编译运行过)
	- macOS 10.14.6 (代码在系统上完成并成功运行)
	- Android API 27 的真实手机 或者 x86_64 的Android API 27 手机模拟器.
	- 其他(待补充)


## 谈谈 Why Swift ? 

 主要最近一段时间接触了移动端的跨平台 (跨 iOS 和 Android 的台), 先后接触了 ReactNative 和 Flutter, 然后就想要不自己来搭建一个跨平台框架? 
 
基于上面的想法有了这篇总结性文章, 上面有一些依赖库以后有时间都可以作为单独篇幅or 系列讲讲(如:  [Swift_Opengl](https://github.com/Guang1234567/Swift_OpenGL/tree/gles32_egl15_android/Sources/SGLOpenGL)).

由于搭建的是跨移动端平台的框架, 但又不想用依赖 VM 的语言.
因为 java, kotlin, dart 不够快 (相对于 swift rust c c++).

好吧其实真正原因是我不懂把 google dalvik 虚拟机魔改成像 `Dart   VM` 这么轻量(相对而言) 并移植到 iOS 上, 常见的有 `Go for Android`,  `Ruby for Android`, `Python for Android`反正我是没听过有 `Go for iOS` .... [社会]

最后的四强出来了: 

- Swift 
- Rust
- C 
- C++

由于不想代码一小时,编译调试一个月, 果断放弃 C C++, 于是最后两强:

- Swift 
- Rust

最先学 Rust, 一个大而全的语言. 

发现Rust 的编码效率不高:

- 由于 rust编译器无法在编译时识别所有情况下堆对象所有权的归属(内存管理依赖于所有权, 谁拥有谁释放), 因此需要引入 `生命周期`概念以及 **相关的语法糖** 来让程序员辅助编译器良好工作, 是不是听起来有点不太对劲, 因为不是 "以人为本" 而是 "以编译器为本" 可想而知编码效率有多低了吧, 连官方文档都说 "Maybe  you must  fight for compiler sometimes"

- 唯一安慰只能是 Rust 还有 "智能指针"(smart pointer) 这玩意作为备胎, 这样一来跟只用 smart-pointer 的 C++ 有啥区别,  当然少了个指针, 拥有函数式范式的语法糖支持 (个人写过 haskell 应该是有发言权的),  满足少数人对 "开源软件" 和 "自由软件" 的向往.

- 唯二安慰有官方提供的 android 交叉编译工具链  rust_android_toolchain_xxx

- 唯三安慰有个"干净宏" 这种元编程方式, 直接改语法树(AST)是有点黑科技啦, 但关键不能断点调试....


最后, 只能是 Swift, 本人崇尚语法简单(类 java c# kotlin c++), 符合人类思维方式的语言, 语法层面内置使用 smart pointer 作为内存管理的方式(引用计数 refCount).

关键还能在 Android 使用 iOS 的生态系统, 上面已经列举的够多了. 

关键还能满足你函数式编程的愿望....
