```
4.1.20

源码解析以理解逻辑为主，异常、校验等代码都屏蔽掉了

NioSocketChannel //异步的客户端TCP连接
NioServerSocketChannel //异步的服务端TCP连接
NioDatagramChannel //异步的UDP连接
NioSctpChannel //异步的客户端SCTP连接
NioSctpServerChannel //异步的服务端SCTP连接

Oio(Old-IO)
OioSocketChannel //同步的客户端TCP连接
OioServerSocketChannel //同步的服务端TCP连接
OioDatagramChannle //同步的UDP连接
OioSctpChannle //同步的客户端SCTP连接
OioSctpServerChannel //同步的服务端SCTP连接
```

```
5处提交一个异步任务添加一个ServerBootstrapAcceptor处理器。该异步任务在init方法之后执行，确保ServerBootstrapAcceptor处理器位于用户设置的handler之后。

1处是ServerBootstrapAcceptor的构造函数，传入的参数是子连接的配置信息，包括childHandler，childGroup，childOptions和childAttrs等。这些信息用于初始化子连接。
2处是channelRead事件处理方法。ServerSocketChannel接收到的消息是从客户端过来的连接channel。然后将childHandler添加到channel的pipeline中，注意设置选项和属性等信息。
3处是将channel注册到childGroup中去。同样也是提交了一个异步任务。监听注册完成的事件，如果注册失败则强制关闭连接，若出现其它异常也会关闭连接。
```

源码解析进度

```
NioServerSocketChannel 实例化过程
NioServerSocketChannel与BossGroup关联
ServerBootstrap#init

ServerBootstrapAcceptor#channelRead
	childGroup.register(child) NioSocketChannel与WorkGroup关联
	
ServerBootstrapAcceptor.channelRead 方法是怎么被调用的呢? 其实当一个 client 连接到 server 时, Java 底层的 NIO ServerSocketChannel 会有一个 SelectionKey.OP_ACCEPT 就绪事件, 接着就会调用到 NioServerSocketChannel.doReadMessages:

接下来就经由 Netty 的 ChannelPipeline 机制, 将读取事件逐级发送到各个 handler 中, 于是就会触发前面我们提到的 ServerBootstrapAcceptor.channelRead 方法啦.
```

![image-20220329145406608](/Users/dlinka/Library/Application Support/typora-user-images/image-20220329145406608.png)

![image-20220329145442932](/Users/dlinka/Library/Application Support/typora-user-images/image-20220329145442932.png)

```
ServerBootstrapAcceptor.channelRead 中会为新建的 Channel 设置 handler 并注册到一个 eventLoop 中, 即:
```

![image-20220329145701364](/Users/dlinka/Library/Application Support/typora-user-images/image-20220329145701364.png)

- 在服务器 NioServerSocketChannel 的 pipeline 中添加的是 handler 与 ServerBootstrapAcceptor.
- 当有新的客户端连接请求时, ServerBootstrapAcceptor.channelRead 中负责新建此连接的 NioSocketChannel 并添加 childHandler 到 NioSocketChannel 对应的 pipeline 中, 并将此 channel 绑定到 workerGroup 中的某个 eventLoop 中.
- 
