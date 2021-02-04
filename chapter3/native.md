在`性能优化分类`这一章节中，介绍了三种性能调优角度。本章将从这三个角度阐述不同的性能优化方式和方法，并提供一定的示例进行补充说明，便于在实践中落地。

通过学习本章节，你将了解到：
- 三个角度可用的优化方式；
- 各类优化方式的实际效果；
- 优化方式与渲染流程间的关系。

下面将开始我们的实践之旅。

# 客户端优化方式

作为React Native的底层容器，性能调优同样采取从底层做起的思路，先从客户端角度的调优方式说起。

## Hermes渲染引擎
React Native的默认引擎目前在iOS平台使用JavaScriptCore（Apple要求），Android平台默认使用JavaScriptCore，也可切换使用V8。

JavaScriptCore本身由词法分析（Lexer）、语法分析（Parser）、字节码生成（ByteCodeGenerator）和解释执行（LLInt & JIT）组成，运行流程如下图：
<img src="https://pic3.zhimg.com/80/v2-8488dc8bf222f987dc548be1321a610e_1440w.jpg" style="zoom:30%" />


大致的执行流程可以概括为：
- 先读取源码文件
- 解析源代码并转换成字节码（bytecode）
- 最后执行

从中可以发现，在运行时解析源码转换字节码是一种时间浪费，如果可以在编译期间生成字节码，一方面可以避免不必要的转换时间，另一方面多出的时间可以用来优化字节码，从而提高执行效率。Facebook 发布的 Hermes 引擎，正是具备该特性的引擎。

Hermes引擎仅支持Android平台，在实践过程中与Bundle包的关系如下图：

[![soSPr6.png](https://s3.ax1x.com/2021/01/22/soSPr6.png)](https://imgchr.com/i/soSPr6)

- 每次新版本的Bundle包被加载时，首次运行都默认使用JavaScriptCore引擎；
- 同时解析源代码并转换成字节码，替换原有的Bundle（源代码）保存在本地；
- 非首次运行且存在字节码文件时，自动使用Hermes引擎运行。

在实践中的效果优异，Android端的性能指标有明显降低，下图使用FMP指标为例：
<img src="https://s3.ax1x.com/2021/01/22/soS4eK.png" style="zoom:50%" />

## 容器热启动
当 Native 打开一个崭新的 React Native 界面时，需要经过如下步骤：

[![so9F4e.png](https://s3.ax1x.com/2021/01/22/so9F4e.png)](https://imgchr.com/i/so9F4e)

其中React Native 容器创建 至 业务代码加载 所用的耗时是FP、FCP和FMP指标的关键因素。

容器热启动的意义在于将界面加载过程中的必经流程提前运行，加快完成渲染页面的速度。启用容器热启动前的流程图：

[![soC8IO.png](https://s3.ax1x.com/2021/01/22/soC8IO.png)](https://imgchr.com/i/soC8IO)


## 容器复用
当多个页面之间存在 ABAB 式的用户行为流时，可以复用React Native容器达到性能调优的目的。用户行为流示意如下图：

<img src="https://pic4.zhimg.com/80/v2-b3abf4d089ccdaa4b1d52bab6c5fbf4f_1440w.png" style="zoom:60%" />


在往复的过程中，Page A和Page B的所处的状态是完全不一致的，Page A全程只被打开了一次，而Page B被打开了两次。原因如下：

- 当Page A在打开Page B时，是作为一个Fragment被Page B遮罩，Page A并没有被关闭。
- Page B在返回Page A时，Page B的Fragment被销毁，其中的React Native容器也一同被销毁了。
- 每次打开Page B时，都将重新创建一个全新的Fragment。

容器复用的优化点在于Fragment及其中的React Native容器可以通过缓存避免被销毁，重复进入时通过复用缓存中的React Native容器来加快页面的展示过程。

使用该种调优方式需要注意下面两个要点：
- 过多的Fragment及其中的React Native容器被缓存时，容易造成内存溢出引起 App Crash
- 复用React Native容器时，会保持上一次会话的全局变量，容易造成业务逻辑错误

应对上述注意要点的方案如下：
- 严格控制缓存中的Fragment数量，采用FIFO的策略自动销毁超出限定数量的Fragment及React Native容器。
- 选择合适的页面变得至关重要，例如静态页面、不存在复杂交互逻辑的页面等。

## Bundle预下载
App启动时，提前下载需要更新的Bundle，仅对 Bundle 提前进行下载操作，并不会解压增量文件。具体流程如下图：

[![soQiLT.png](https://s3.ax1x.com/2021/01/22/soQiLT.png)](https://imgchr.com/i/soQiLT)

使用该方案需要解决如下问题：
- 需要更新Bundle的页面数量较多时，若无差别使用预下载时容易造成网络下载列队阻塞。
- Bundle 存在下载失败的概率，会丧失预下载的想要达到的效果。

上述问题通过继续深度优化解决：
- 预下载的时机需要符合如下几个条件：
    - 利用Native底包优势，在Native页面时开启预下载；
    - 业务改动频率较低，如静态页面；
    - 页面具备一定的停留时长。
- 采取优先级异步多线程下载策略，按不同维度设定优先级，如 Bundle 使用率。
- 建立重试机制，采用轮询的方式重新下载 之前下载失败的Bundle。


## Bundle预加载
在 React Native 容器创建之前，解压 Bundle 文件并完成差分增量更新操作，通常配合 React Native 容器热启动和 Bundle 预下载使用。
Bundle 加载主要完成下述几项工作：

1. 更新Bundle文件
2. 编译JS代码
3. 执行JS代码

当这些工作被提前执行，会使页面在打开时更快得进入到渲染阶段，降低FCP、FMP和TTI的耗时。下图为配合容器热启动和 Bundle 预下载使用时的页面加载时序：

[![sjFNaF.png](https://s3.ax1x.com/2021/01/26/sjFNaF.png)](https://imgchr.com/i/sjFNaF)


同时，随着 React Native 容器采用 Hermes 引擎，Bundle 被打包为单个文件，相比使用 JSCore 被打包成多个文件具有额外两项优势：
- 更新 Bundle 文件阶段，单文件的更新速率优于多文件；
- 编译JS代码阶段，单文件减少了多个文件加载耗时。

## Sync API
React Native 与 Native 之间采用异步通信机制，当线程繁忙时，会产生阻塞和等待。

另外，在首屏渲染过程中，内存获取数据较慢的场景也会出现，耗时可能高达200ms。

解决上述问题，主要有以下几个方向：
- 对内存读写数据类 API
- Sync API 耗时可控在毫秒级
- Chrome Dev 不支持 Sync，需特殊处理
- 有利于解决阻塞依赖 Native 异步接口调用的场景

此时，使用 Sync 同步方案显得可行，可解决如下场景：
- 获取 ABTesting 实验号
- 获取本地 Storage 内容
- 获取功能开关列表
- 获取屏幕 Size
- SOTPCookie

## Hermes Bundle
在切换为Hermes渲染引擎的情况下，渲染性能得到明显的提升。但通过Hermes渲染引擎这一小节的了解，我们知道使用Hermes时，首次使用的仍然是JSCore引擎，只有当字节码文件编译完成后才能切换为Hermes引擎。

鉴于这个前提条件，当Bundle为最新版本时，首次若想使用Hermes引擎必须具备两个基本条件：
- App Size 拥有足够余量
- Bundle内容为字节码

当Bundle内容为字节码时，需要切换的不仅仅是引擎部分，还有配套的打包及差分增量的功能。同时由于字节码的Size通常是源码Size的四倍，也就从App Size维度限定了该性能调优模式的使用规模。

使用Hermes Bundle时，实践过程与Bundle包的关系如下：

[![sjESET.png](https://s3.ax1x.com/2021/01/26/sjESET.png)](https://imgchr.com/i/sjESET)

可以发现，使用Hermes Bundle之后，执行过程与原来JSC的执行过程相差无几。

通常，建议对业务流程中第一个React Native Bundle采用该种优化方式。原因在于首个React Native Bundle相对更贴近于Native层，可使用的优化手段也相对较少。