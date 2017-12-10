# Reactor模式详解

## 什么是Reactor模式

> Reactor模式是事件驱动的，有一个或多个并发输入源，有一个Service Handler，有多个Request Handlers，这个Service Handler会同步将输入的请求（Event）多路复用的分发给相应的Request Handler。

如图：

![](http://www.blogjava.net/images/blogjava_net/dlevin/Reactor_Simple.png)

从结构上来看，Reactor模式与消费者模式类似，有多个生产者将事件放入Queue中，然后多个消费者**主动**从Queue中Poll事件来处理。

然而不同点在于，Reactor模式中并没有Queue的缓冲，而是Event输入后，Service Handler会根据Event的类型自动发放到相应的Request Handler处理。

## Reactor模式结构

![](http://www.blogjava.net/images/blogjava_net/dlevin/Reactor_Structures.png)

**Handle**：操作系统的句柄，是对资源在操作系统层面上的一种抽象。它可以使打开的文件，一个连接（Socket），Timer等。Reactor模式中，通常指Socket Handle，即一个网络连接（Connection，在Java NIO中的channel），这个channel在注册到`Synchronous Event Demultiplexer`中，以监听Handle中发生的时间，对ServerSocketChannel可以使CONNECT时间，对SocketChannel可以使READ、WRITE、CLOSE事件等。

**Synchronous Event Demultiplexer**：阻塞等待一系列Handle中的时间的到来，如果阻塞等待返回，即表示再返回的Handle中可以不阻塞的执行返回的事件类型。这个模块一般使用操作系统的select实现。在Java NIO中用Selector来封装，当Selector.select()返回时，可以调用Selector的selectedKeys()方法获取Set<SelectionKey>，一个SelectKey表达一个有事件发生的Channel以及该Channel上的事件类型。上图中的“Synchronous Event Demultiplexer --notifies--> Handle”的流程如果是对的，那内部实现应该是select()方法在事件到来后会先设置Handle的状态，然后返回。

**Initiation Dispatcher**：用于管理Event Handler，即Event Handler的容器，用以注册、移除Event Handler等；另外，它还作为Reactor模式的入口调用Synchronous Event Demultiplexer的select方法以阻塞等待事件返回，当阻塞等待返回时，根据事件发生的Handle将其分发给对应的Event Handler处理，即回调Event Handler中的handle_event()方法。

**Event Handler**：定义事件处理方法：handle_event()，以供InitiationDispatcher回调使用。

**Concrete Event Handler**：事件EventHandler接口，实现特定事件处理逻辑。

## Reactor模式模块之间的交互

Reactor各个模块之间的交互流程，放上序列图：

![](http://www.blogjava.net/images/blogjava_net/dlevin/Reactor_Sequence.png)

1. 初始化InitiationDispatcher，并初始化一个Handle到EventHandler的Map。
2. 注册EventHandler到InitiationDispatcher中，每个EventHandler包含对相应Handle的引用，从而建立Handle到EventHandler的映射（Map）。
3. 调用InitiationDispatcher的handle_events()方法以启动Event Loop。在Event Loop中，调用select()方法（Synchronous Event Demultiplexer）阻塞等待Event发生。
4. 当某个或某些Handle的Event发生后，select()方法返回，InitiationDispatcher根据返回的Handle找到注册的EventHandler，并回调该EventHandler的handle_events()方法。
5. 在EventHandler的handle_events()方法中还可以向InitiationDispatcher中注册新的Eventhandler，比如对AcceptorEventHandler来，当有新的client连接时，它会产生新的EventHandler以处理新的连接，并注册到InitiationDispatcher中。

## Reactor模式实现

Logging Server中的Reactor模式实现分两个部分：Client连接到Logging Server和Client向Logging Server写Log。因而对它的描述分成这两个步骤。

### Client连接到Logger Server

![](http://www.blogjava.net/images/blogjava_net/dlevin/Reactor_LoggingServer_connect.png)

1. Logging Server注册LoggingAcceptor到InitiationDispatcher。
2. Logging Server调用InitiationDispatcher的handle_events()方法启动。
3. InitiationDispatcher内部调用select()方法（Synchronous Event Demultiplexer），阻塞等待Client连接。
4. Client连接到Logging Server。
5. InitiationDisptcher中的select()方法返回，并通知LoggingAcceptor有新的连接到来。 
6. LoggingAcceptor调用accept方法accept这个新连接。
7. LoggingAcceptor创建新的LoggingHandler。
8. 新的LoggingHandler注册到InitiationDispatcher中(同时也注册到Synchonous Event Demultiplexer中)，等待Client发起写log请求。


### Client向Logging Server写Log

![](http://www.blogjava.net/images/blogjava_net/dlevin/Reactor_LoggingServer_log.png)

1. Client发送log到Logging server。
2. InitiationDispatcher监测到相应的Handle中有事件发生，返回阻塞等待，根据返回的Handle找到LoggingHandler，并回调LoggingHandler中的handle_event()方法。
3. LoggingHandler中的handle_event()方法中读取Handle中的log信息。
4. 将接收到的log写入到日志文件、数据库等设备中。3.4步骤循环直到当前日志处理完成。
5. 返回到InitiationDispatcher等待下一次日志写请求。

## EventHandler接口定义

对EventHandler的定义有两种设计思路：single-method设计和multi-method设计：

**single-method**：它将Event封装成一个Event Object，EventHandler只定义一个handle_event(Event event)方法。这种设计的好处是有利于扩展，可以后来方便的添加新的Event类型，然而在子类的实现中，需要判断不同的Event类型而再次扩展成 不同的处理方法，从这个角度上来说，它又不利于扩展。另外在Netty3的使用过程中，由于它不停的创建ChannelEvent类，因而会引起GC的不稳定。

**multi-method**：这种设计是将不同的Event类型在EventHandler中定义相应的方法。这种设计就是Netty4中使用的策略，其中一个目的是避免ChannelEvent创建引起的GC不稳定， 另外一个好处是它可以避免在EventHandler实现时判断不同的Event类型而有不同的实现，然而这种设计会给扩展新的Event类型时带来非常 大的麻烦，因为它需要该接口。


## 参考博文

[Reactor模式详解](http://www.blogjava.net/DLevin/archive/2015/09/02/427045.html)


