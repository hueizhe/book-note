# netty

## netty 的核心组件

* Channel
* Callback
* Future
* 事件和 ChannelHandler

## Channel
  Channel 是 Java NIO 的一个基本构造
  >它代表一个到实体的开发连接，如读操作和写操作。实体指 一个硬件设备，一个文件，一个网络套接字或者一个能够执行
  一个或者多个不同的I/O操作的程序组件

目前，可以把Channel看作是传入或者传出数据的载体。因此，它可以被打开或者被关闭，连接或者被断开
  
## 回调
   一个回调其实就是一个方法，一个指向已经被提供给另外一个方法的 方法的引用。这使得后者可以在适当的时候调用前者
   
## Future
  * Future 提供了另一种在操作完成时通知应用程序的方式。这个对象可以看作是一个异步操作的结果的占位符；
  他将在未来的某个时刻完成，并提供对其结果的访问
  
  * 每个netty 的出站I/O 操作都将返回一个ChannelFuture;也就是说它们都不会阻塞。 netty完全是异步和事件驱动的
  
  * ChannelFutureListener 提供的通知机制消除了手动检查对应的操作是否完成的必要
  
  * 回调和Future 是相互补充的机制， 他们互相结合，构成netty本身的关键模块之一
  
## 事件和ChannelHandler

 netty 使用不同的事件来通知我们状态的改变 或者是操作的状态。 这使得我们能够基于已经发生的事件来触发适当的动作
 这些动作可能是：
 * 记录日志
 * 数据转换
 * 流控制
 * 应用程序逻辑
 
 Netty 是一个网络编程框架， 所以事件是按照它们与入站或者出站 **数据流**的相关性进行分类的。
 
 可能由入站数据或者相关的状态更改而触发的事件包括：
 * 连接已被激活或者连接失活
 * 数据读取
 * 用户事件
 * 错误事件
 
 出站事件是未来将会触发的某个动作的操作结果， 这些动作包括：
 * 打开或者关闭到远程节点的连接
 * 将数据写到或者冲刷到套接字
 
 每个事件都可以被分发给ChannelHandler类中某个用户实现的方法。这是一个很好的将事件驱动范式直接转换为应用程序构件块的例子
 
 ## Future 、回调 和 ChannelHandler
Netty 的异步编程模型是建立在Future和回调的概念之上的，而将事件派发到ChannelHandler的方法则发生在更深的层次上。
 
结合在一起，这些元素就提供了一个处理环境，使你的应用程序逻辑可以独立于任何网络操作相关的顾虑而独立地演变。

拦截操作以及高速地转换入站数据和出站数据， 都只需要你 提供回调或者利用操作所返回的Future. 这使得链接变得既简单又高效，
并且促进了可重用的通用代码的编写

## 选择器  事件和 EventLoop

Netty 通过触发事件将Selector 从应用程序中抽象出来，消除了所有本来将需要手动编写的派发代码。在内部， 将会为每个Channel
分配一个EventLoop, 用以处理所有事件，包括：
* 注册感兴趣的事件
* 将事件派发给ChannelHandler
* 安排进一步的动作

EventLoop 本身只由一个线程驱动， 其处理了一个Channel 的所有I/O事件，并且在该EventLoop的整个生命周期内都不会改变


## Channel、 EventLoop 和ChannelFuture
* Channel -- Socket
* EventLoop -- 控制流，多线程处理、并发
* ChannelFuture -- 异步通知

## ChannelHandler 和 ChannelPipeline



## 零拷贝

零拷贝（zero-copy）是一种目前只有在使用 NIO 和 Epoll 传输时才可使用的特性。它使你可以快速
高效地将数据从文件系统移动到网络接口， 而不需要将其从内核空间复制到用户空间，其在像 FTP 或者
HTTP 这样的协议中可以显著地提升性能。 但是， 并不是所有的操作系统都支持这一特性。特别地，它对
于实现了数据加密或者压缩的文件系统是不可用的——只能传输文件的原始内容。 反过来说，传输已被
加密的文件则不是问题


## Channel 的生命周期

| 状态              |   描述                                                                      |
|-------------------| --------------------------------------------------------------------------- |
|ChannelUnregistered| Channel已经被创建，但还未注册到EventLoop                                    |
|ChannelRegistered  | Channel 已经被注册到了EventLoop                                             |
|ChannelActive      | Channel 处于活动状态（已经连接到它的远程节点）。他现在可以接收和发送数据    |
|ChannelInactive    | Channel 没有连接到远程节点                                                   |

Channel 的正常生命周期
 ChannelRegistered   ----> ChannelActive
                                |
                                |
 ChannelUnregistered <---- ChannelInactive