# Node.js 其实可以很简单
我在16年初的时候第一次接触 **Node.js**，虽然看着官网的教程、api，也能写起来代码，其实心里很慌的，也是一脸懵，因为啥啥都不理解。再次特别感谢 @朴灵 的《深入浅出 node》这本书，以及 《你不知道的 js》，多数开发者这两本书看完，**Node.js** 都可以玩的6了。当然你也可以先听我娓娓道来。

Node 官网有这么一段话
```javascript
Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient. Node.js' package ecosystem, npm, is the largest ecosystem of open source libraries in the world.
```
这段话可以说包裹了Node的全部特性，如果提取几个关键词：**v8、事件循环、非阻塞i/o，npm**。是不是都很常见，那我们往下看。

## 什么是 Node.js
其实 **Node.js** 并不是一门语言，也不是什么 web service 框架，你在它的官网也不会找到这些词。它其实就是一个运行环境，一个平台，构建在大名鼎鼎的 v8 执行引擎之上，有点像 java 的 jvm（jvm 是java虚拟机，运行编译之后的calss文件，jvm 和 **Node.js** 区别非常大，这里只是把它们两作为运行环境的部分相比较）。也就是说 **Node.js** 提供了一个 javascript 的运行环境，js 作为一门浏览器脚本语言，之所以能运行在并没有安装浏览器的服务器上都是 **Node.js** 干的好事。

所以你本质上写的都还是 js 代码，这也是为什么我会推荐《你不知道的 js》这套书（上中下三本，其实我只看过下，因为 js 的闭包才是 **Node.js** 选择它的关键）。

## Node 架构

![node framework](../public/img/node-framework.jpeg)
从图中我们可以看出 **Node.js** 的三层架构。

* 最上层，**Node.js** 标准库，这部分是由 Javascript 编写的，即我们使用过程中直接能调用的 API。在源码中的 lib 目录下可以看到。
* Node Bindings 是沟通 js 和 c++ 的关键，封装下层 c/c++ 实现细节，向上层提供基础API服务，上下层通过 bindings 交换数据。
* 最下层是支撑 **Node.js** 运行的关键，由 C/C++ 实现。
    * V8：Google 推出的 Javascript VM，也是 **Node.js** 为什么使用的是 Javascript 的关键，它为 Javascript 提供了在非浏览器端运行的环境，它的高效是 **Node.js** 之所以高效的原因之一。
    * Libuv：它为 **Node.js** 提供了跨平台，线程池，事件池，异步 I/O 等能力，是**Node.js** 如此强大的关键。
    * C-ares：提供了异步处理 DNS 相关的能力。
    * http_parser、OpenSSL、zlib 等：提供包括 http 解析、SSL、数据压缩等其他的能力。

## Libuv

libuv 的官网描述很简单：它是一个高性能的，事件驱动的I/O库，主要应用于 **Node.js**，它使用 c 编写。它的前身是linux上的libev，专门封装linux上的内核消息机制。后来**Node.js**重写了它，并在Windows上使用iocp技术重新实现了一遍。所以**Node.js**现在能跨平台运行在Windows上了。

![libuv framework](../public/img/libuv-framework.png)

从图中可以看出，对于 Network I/O和以File I/O为代表的另一类请求，异步处理的底层支撑机制是完全不一样的

对于Network I/O相关的请求， 根据OS平台不同，分别使用Linux上的epoll，OSX和BSD类OS上的kqueue，SunOS上的event ports以及Windows上的IOCP机制。

而对于File I/O为代表的请求，则使用thread pool。利用thread pool的方式实现异步请求处理，在各类OS上都能获得很好的支持。

也就是说，虽然我们使用 js 书写代码，但是由 C/C++ 来执行真正的系统调用，所以效率是很高的。

## 异步、非阻塞 I/O
这里的异步主要实现方式就是回调函数，非阻塞 I/O 是因为 **Node.js** 的底层 I/O 是使用 libuv 进行的，而 libuv 每次都会创建线程处理 I/O 请求，并且在处理完成后，通过回调函数将结果返回给调用层。看到了吧，真正进行 I/O 操作的时候并不是单线程，所谓的 **Node.js** 单线程，指的是 **Node.js** 中 js 运行环境是单线程，**Node.js** 并没有给 js 执行时创建新线程的能力。

* 一个异步 I/O 的大致流程如下
    * 用户通过 Javascript 代码调用 **Node.js** 核心模块，将参数和回调函数传入到核心模块
    * **Node.js** 核心模块会将传入的参数和回调函数封装成一个请求对象
    * 将这个请求对象推入到 I/O 线程池等待执行
    * Javascript 发起的异步调用结束，Javascript 线程继续执行后续操作

* 执行回调
    * I/O 操作完成后，会将结果储存到请求对象的 result 属性上，并发出操作完成的通知
    * 每次事件循环时会检查是否有完成的 I/O 操作，如果有就将请求对象加入到 I/O 观察者队列中，之后当做事件处理
    * 处理 I/O 观察者事件时，会取出之前封装在请求对象中的回调函数，执行这个回调函数，并将 result 当参数，以完成 Javascript 回调的目的

这也就是为什么单线程的语言 js ，能在 **Node.js** 里面实现异步操作的原因，因为真正耗时的 I/O 操作在底层还是通过线程池运行。

## 事件循环
上面提到了事件循环，就是不断得等待事件的发生，然后将这个事件的所有处理器，以它们订阅这个事件的时间顺序，依次执行。当这个事件的所有处理器都被执行完毕之后，事件循环就会开始继续等待下一个事件的触发，不断往复。

当同时并发地处理多个请求时，以上的概念也是正确的，可以这样理解：在单个的线程中，事件处理器是一个一个按顺序执行的。

即如果某个事件绑定了两个处理器，那么第二个处理器会在第一个处理器执行完毕后，才开始执行。在这个事件的所有处理器都执行完毕之前，事件循环不会去检查是否有新的事件触发。在单个线程中，一切都是有顺序地一个一个地执行的！

libuv 事件循环的核心函数是 uv_run，**Node.js** 会一直调用 uv_run 直到到循环不在存活。

## 总结
**Node.js** 其实就是 libuv 的一个应用而已。简单的说 **Node.js** 只是把JavaScript解释成C++的回调，并挂在libuv消息循环上，等待处理。这样就实现了非阻塞的异步处理机制。

因为 **Node.js** 异步的核心是通过回调完实现的，而 JavaScript 的闭包，非常适合做回调函数。因为我们一般都希望当回调发生时，闭包能记住它原来所在的执行上下文，闭包不就是干这事的嘛。这里我就不知道到底是 **Node.js** 选择了 js ，还是 **Node.js** 为了迎合 js 选择了异步回调这种机制。

## Node 优缺点

* 优点
    * 高并发（最重要的优点）
    * 适合I/O密集型应用（静态资源服务器、blog、聊天室）

* 缺点
    * 不适合CPU密集型应用（模板渲染、压缩/解压缩、加/解密）；CPU密集型应用给 **Node.js** 带来的挑战主要是：由于JavaScript单线程的原因，如果有长时间运行的计算（比如大循坏），将会导致CPU时间片不能释放，使得后续I/O无法发起；解决方案：分解大型运算任务为多个小任务，使得运算能够适时释放，不阻塞I/O调用发起
