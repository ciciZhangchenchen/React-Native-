# React-Native-
React Native学习笔记
## JavaScript Core
RN 中的核心部分之一, RN 的核心驱动力就来自 JS Engine, 我们所有的 JS 代码都会通过 JS Engine 来编译执行, 包括 React 的 JSX 也离不开 JS Engine, JavaScript Core 是其中一种 JS 引擎, 还有 Google 的 V8 引擎, Mozilla 的 SpiderMonkey 引擎

## RN, React, JavaScript Core 的关系
### react

React 是一个纯 JS 框架, 所有的代码都要用 JS Engine 来解释执行, 而且 React 里面用 JS 实现了 Virtual Dom, 实现了数据驱动编程的模式, 而且 React 还提出了独特的 JSX 语法, 实现了在 JS 就可以写 HTML 和 CSS 代码, 配合虚拟 DOM 就实现了基于 MVC 的前端框架

关于 Virtual Dom 的 Diff 算法是 React 知名的技术点, 他利用很巧妙的算法实现了虚拟 DOM 快速对比新旧节点的不同从而节省了原生 DOM 重复渲染的性能浪费

React 无法接触到底层的一些事情, 比如说驱动, 声卡, 显卡, 线程, 进程等基于操作系统底层的东西

### RN

RN 情况比较复杂, RN 的原生代码驱动 JS Engine, 然后 JS Engine 解析执行 React 或者相关 JS代码, 然后把计算好的结果返回给 Native Code, 然后在进行驱动设备上的硬件

RN 利用 Virtual Dom 和数据驱动编程简化开发, 不由浏览器去绘制 UI 控件, 最终是由原生UI控件去负责, 保证了原生的体验, 因为 JavaScript 是不需要编译的, 所以 RN 的运行效率会比其他基于 HTML5、CSS3 等技术的 PhoneGap、AppCan 高很多, 这些技术直接使用 HTML5 进行渲染, 都会用到大量的 DOM 技术, 是很耗时的操作

> 所以 React Native = JavaScript Core + React.js + Bridges

## RN 架构分析
![image](https://user-images.githubusercontent.com/23628263/171083958-02e20663-48c3-4d7f-86f4-d9275625c844.png)
- Java层: 主要负责 Native 的 UI 渲染和底层功能调用, Java 层的核心 jar 包是 react-native.jar, 封装了很多接口, 例如 Module, Registry, Bridge
- C++层: 主要封装了 JavaScriptCore, 起到了解析 JS 代码的作用
- JS 层: 利用 JS 代码去进行事件的分发和 UI 代码的编写, 主要以下几部分:

  Component: 编写 JSX 来构建虚拟 DOM
  
  Lifecycle: 生命周期必不可少, 可以在组件的各种时期器进行一些操作
  
  Layout: 使用 FlexBox 进行布局, 用 css-layout, 不依赖 DOM, 可以编译成 Native 端代码, 用于布局样式的展示, 目前不支持 CSS3
  
## Bridge

Bridge 顾名思义就是 JS 和 Native 通信的一个桥梁, 所有的本地存储、图片资源访问、图形绘制、3D加速、网络访问、震动效果、NFC、原生控件绘制、地图、定位、通知等等很多功能都是由 Bridge 封装成 JS 接口以后注入 JS Engine 供 JS 调用

每一个支持 RN 的原生功能必须有同一个原生模块和一个 JS 模块, JS 模块方便调用其接口, 原生模块去根据接口调用原生功能实现原生效果

Bridge 原生代码负责管理原生模块并能够方便的被 React 调用, 每个功能 JS 封装主要是对 React 做适配, JS 和 Native 之间不存在任何指针传递, 所有的参数均由字符串传递

## Bridge 各模块介绍

### RCTRootView
- RCTRootView 是 RN 加载的地方, 从这里开始有了 JS Engine. JS 代码被加载进来, 对应的原生模块也被加载进来, 然后 JS loop 开始
- RCTRootView 初始化代码完成后, 整个 React Native 运行环境就已经初始化好, JS 代码也加载完毕, 所有 React 的绘制都会有这个 RCTRootView 来管理
- 创建并持有 RCTBridge
- 加载 JS Bundle 初始化 JS 运行环境
- 初始化 JS 运行环境后会在 app 显示 loadingView
- JS 运行环境准备好就被加载视图 RCTRootContentView 替换加载视图
- 准备工作就绪之后就调用 AppRegistry.runApplication 正式启动 RN 代码, 从 Root 组件开始 UI 绘制

### RCTBridge

- 加载和初始化专用类, 用于前期 JS 的初始化和原生代码的加载
- 负责加载各个 Bridge 模块供 JS 调用
- 找到并注册实现了 RCTBridgeModule protocol 的类
- 创建持有 RCTBatchedBridge

### RCTBatchedBridge

负责 Native 和 JS 之间的相互调用, 也就是信息通信

### RCTJavaScriptLoader

实现远程代码的核心, 热更新, 开发环境代码加载. 静态 jsbundle 加载

###  RCTContextExecutor

封装了 JS 和 Native 代码的互相调用逻辑

### RCTModuleData

加载管理所有与 JS 交互的原生代码, 把交互代码封装成 JS 模块

### RCTModuleMethod

记录所有原生代码的导出函数地址, 同时生成对应的字符串映射到改函数地址
翻译所有 J2N call
### MessageQueue

RN 是不用 JS 引擎的 UI 渲染控件的, 但是会用到 JS 引擎的 DOM 操作管理能力来管理所有 UI 节点, 每次在写完 UI 组件代码后会交给 yoga 去做布局排版, 然后调用原生组件绘制
MessageQueue 负责跳出 JS 引擎, 记录原生接口的地址和对应的 JS 函数名, 然后在 JS 调用该函数的时候把调用转发给原生接口

### Bridge 如何工作

- JS 和 IOS 通信用的是 JavaScript Core
- JS 和 Android 通信用的是 Hermes
### RN 现在主要有 3 个线程
- JS Thread 执行线程, 负责逻辑层面的处理, Metro 将 React 源码打包成 bundle 文件, 然后传给 JS 引擎执行, 现在 IOS 和 Android 统一的是 JSC
- UI Thread 主要主责原生渲染 Native UI 和调用原生能力 (Native Module)
- Shadow Thread 这个线程主要创建 Shadow Tree 来模拟 React 结构树, RN 使用 Flexbox 布局, 但是原生不支持, Yoga 引擎就是将 Flexbox 布局转换为原生布局的
###  一些需要了解的基础概念

- UIManager: 在 Native 里只有它才有权限调用客户端UI
- JS Thread: 运行打包好的 bundle 文件, 这个文件就是我们写完代码去进行打包的文件, 包含了业务逻辑, 交互和模块化组件
- Shadow Node: Native 的一个组件树, 可以监听 App 里的 UI 变化, 类似于虚拟 DOM 和 DOM
- Yoga: Fackbook 开源的一个布局引擎, 用来把 Flexbox 的布局转换为 Native 的布局
### 打开App会发生什么
怎样创建 View 组件?

我在官方项目上index.js加上以下代码就可以看到 Bridge 运行的过程

import MessageQueue from 'react-native/Libraries/BatchedBridge/MessageQueue.js'

MessageQueue.spy(true, msg => console.log(msg))
然后在 App.js 加入以下代码

<View style={{
  backgroundColor: 'green',
  width: 100,
  height: 100
}}/>

- 用户点击 App 图标
- UIManager 线程: 加载所有 Native 库和 Native 组件比如 Images, View, Text
- 告诉 JS 线程, Native 部分准备好了, JS 侧开始加载 bundle 文件
- JS 通过 Bridge 发送一条 Json 数据到 Native , 告诉 Native 怎么创建 UI, 所有的 Bridge 通信都是异步的, 为了避免阻塞 UI
- Shadow 线程最先拿到消息, 创建 UI 树
- Yoga 引擎获取布局并转为 Native 的布局
- 之后 UI Manager 执行一些操作展示 Native UI

![image](https://user-images.githubusercontent.com/23628263/171088284-d7ba93a1-5858-47bd-82d0-f84ef6ca8cd5.png)

JS thread会先对其序列化,形成下面一条消息

UIManager.createView([343,"RCTView",31,{"backgroundColor":-16181,"width":200,"height":200}])
通过Bridge发到ShadowThread。Shadow Tread接收到这条信息后，先反序列化，形成Shadow tree，然后传给Yoga，形成原生布局信息。

接着又通过Bridge传给UI thread。

UI thread 拿到消息后，同样先反序列化，然后根据所给布局信息，进行绘制。

从上面过程可以看到三个线程的交互都是要通过Bridge，因此瓶颈也就在此。


### Brige的缺点

- 有两个不同的领域 JS 和 Native, 他们彼此之间不能相互感知, 也并不能共享相同内存
- 通信基于 Bridge 的异步通信, 所以并不能保证数据百分百及时传达到另一侧
- JSON 传输大数据非常慢, 内存不能共享, 所有的传输都是新的复制
- 序列化。通过JSON格式来传递消息，每次都要经历序列化和反序列化，开销很大。
- 
- 无法同步更新 UI, 比方在渲染列表的时候, 滑动大量加载数据, 屏幕可能会发生卡顿或闪烁
- RN 代码仓库很大, 库比较重, 所以修复 Bug 和开源社区贡献代码的效率也相应更慢
因此解决 Bridge 的缺点主要遵从三个原则

- JS 和 Native 不用通信, 或者直接不走 Bridge
- JS 和 Native 减少通信, 在两端无法避免的情况下, 避免通信减少次数, 多个请求可以合成一个
- 减少 JSON 的大小

对大多数 React Native 应用来说，业务逻辑是运行在 JavaScript 线程上的。这是 React 应用所在的线程，也是发生 API 调用，以及处理触摸事件等操作的线程。更新数据到原生支持的视图是批量进行的，并且在事件循环每进行一次的时候被发送到原生端，这一步通常会在一帧时间结束之前处理完（如果一切顺利的话）

### RN 新架构 (2018年 RN0.59还没有完成，RN0.64完成)
Since React Native 0.64, Hermes also runs on iOS

> Facebook 团队也正在重新架构整个 RN 框架代码

下面来解释一下这些新概念:

1. JSI 将会替换 Bridge 是一个 JS 引擎接口, 它是 JavaScript Interface 的缩写, 一个用 C++ 写成的轻量级框架, 作用就是用过 JSI, JS 可以直接获取 C++ 对象引用, 然后调用对应方法, 为了让 JS 和 Native 相互感知, 不需要在通过 JSON 数据传输通信, 将允许 Native 对象被导出成 JS 对象, 反过来也可以, 相互通信的方式也会变为同步, 架构的其他部分是基于这个的, JSI 的另一优势是磨平了 JS 引擎的差异, 使用 JSI 不需要关心 JavaScript Core 还是 Hermes, JSI 底层都消化了, 因此只要基于 JSI 的接口编写即可

2. Fabric 是 UI Manager 的新名称, 将负责 Native UI 渲染, 和当前 Bridge 不同的是, 可以通过 JSI 导出自己的 Native 函数, 在 JS 层可以直接使用这些函数引用, 反过来 Native 可以直接调用 JS 层, 从而实现同步调用, 这带来更好的数据传输和性能提升, 另外一个好处就是 Fabric 也支持渲染优先级, 比如 React 里的 Concurrent 和 Suspense 模式

原来的通信模型:

![image](https://user-images.githubusercontent.com/23628263/171088451-085d58d0-afde-479a-a873-d007b3faa734.png)

改变架构后的通信模型:

![image](https://user-images.githubusercontent.com/23628263/171088469-dd5b596a-f78c-4391-9067-4a66838a55a4.png)

3. CodeGen 为了让 JS 测成为两端通信的唯一来源, 可以让开发者创建 JS 的静态类, 以便 Native 端可以识别他们, 并且避免每次都校验数据, 将会提升性能, 减少数据传输出错的可能性, 自动将 Flow 或 TS 有静态类型检查的 JS 代码转换为 Fabric 和 TurboModules 使用的原生代码

4. Turbo Modules, Native 组件的新名字就是 Turbo Modules, 组件的作用是相同的, 但是实现和行为会不同, 这些组件是懒加载的, 而现在启动时就会全部加载, 他们也是 JSI 导出的, 所以 JS 可以拿到这些组件的引用, 并且在 RN 里使用他们, 启动的时候会带来更好的性能表现, 比如说可以通过 JSI 直接调用 Native 模块实现同步操作, 调用本地原生能力

5. Lean Core 对 RN 库架构的变化, 目的减轻 lib 的负担, 并帮助社区更好解决问题

### 回顾打开 App 会发生什么
- 点击 App 图标
- Fabric 加载 Native 侧
- 然后通知 JS 线程 Native 侧准备好了, JS 侧会加载所有的 bundle JS 文件, 里面包含了所有的 JS 和 React 逻辑组件
- JS 通过一个 Native 函数的引用, 调用到 Fabric, 同时 Shadow Node 创建一个和以前一样的 UI 树
- Yoga 进行布局计算, 把基于 Flexbox 的布局转化为 Native 端的布局
- Fabric 执行操作并显示 UI
- 没有了 Bridge 提升了性能,可以用同步的方式进行操作, 启动时间也快, App 也将更小



