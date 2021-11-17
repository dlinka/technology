### 关键类

#### BrokerController

```java
public class BrokerController {
		private MessageStore messageStore;
    private RemotingServer remotingServer;
    private RemotingServer fastRemotingServer;
	  private ExecutorService sendMessageExecutor;
}
```

#### DefaultMessageStore

```java
public class DefaultMessageStore implements MessageStore {
		private final CommitLog commitLog;
	  private final AllocateMappedFileService allocateMappedFileService;
}
```

#### CommitLog

```java
public class CommitLog {
		protected final MappedFileQueue mappedFileQueue;  
    protected final PutMessageLock putMessageLock;
}
```

#### MappedFileQueue

```java
public class MappedFileQueue {
  	//MappedFile就是commitlog文件
		private final CopyOnWriteArrayList<MappedFile> mappedFiles = new CopyOnWriteArrayList<>();
}
```

---

#### 1.BrokerStartup#main

```java
public static void main(String[] args) {
		start(createBrokerController(args));
}
↓
↓
public static BrokerController createBrokerController(String[] args) {
  	final BrokerController controller = new BrokerController(
        brokerConfig,
        nettyServerConfig,
        nettyClientConfig,
        messageStoreConfig);
		controller.initialize(); //2
  	return controller;
}
```

#### 2.BrokerController#initialize

```java
public boolean initialize() throws CloneNotSupportedException {
    //创建DefaultMessageStore
    this.messageStore = new DefaultMessageStore(this.messageStoreConfig, this.brokerStatsManager, this.messageArrivingListener, this.brokerConfig); //3

    //创建NettyRemotingServer
    this.remotingServer = new NettyRemotingServer(this.nettyServerConfig, this.clientHousekeepingService);
  
    //接收消息写入的线程池
    this.sendMessageExecutor = new BrokerFixedThreadPoolExecutor(
				this.brokerConfig.getSendMessageThreadPoolNums(),
				this.brokerConfig.getSendMessageThreadPoolNums(),
        1000 * 60,
        TimeUnit.MILLISECONDS,
        this.sendThreadPoolQueue,
        new ThreadFactoryImpl("SendMessageThread_"));

    this.registerProcessor(); //4
}
```

#### 3.DefaultMessageStore的构造方法

```java
public DefaultMessageStore(final MessageStoreConfig messageStoreConfig, final BrokerStatsManager brokerStatsManager, final MessageArrivingListener messageArrivingListener, final BrokerConfig brokerConfig) throws IOException {
    //创建AllocateMappedFileService
    this.allocateMappedFileService = new AllocateMappedFileService(this);

    //创建CommitLog
    this.commitLog = new CommitLog(this); //3.1

    this.allocateMappedFileService.start(); //3.2
}
```

#### 3.1.CommitLog的构造方法

```java
public CommitLog(final DefaultMessageStore defaultMessageStore) {
		this.mappedFileQueue = new MappedFileQueue(
      	defaultMessageStore.getMessageStoreConfig().getStorePathCommitLog(),
				defaultMessageStore.getMessageStoreConfig().getMappedFileSizeCommitLog(),
      	defaultMessageStore.getAllocateMappedFileService());
  
  	this.putMessageLock = defaultMessageStore.getMessageStoreConfig().isUseReentrantLockWhenPutMessage() ? new PutMessageReentrantLock() : new PutMessageSpinLock();
}
```

#### 3.2.AllocateMappedFileService#run

```java
public void run() {
		while (!this.isStopped() && this.mmapOperation()) {

		}
}
↓
↓
```

#### 4.BrokerController#registerProcessor

```java
public void registerProcessor() {
    //消息接收处理类
    SendMessageProcessor sendProcessor = new SendMessageProcessor(this);

		this.remotingServer.registerProcessor(RequestCode.SEND_MESSAGE_V2,
                                          sendProcessor,
                                          this.sendMessageExecutor); //4.1
}
```

#### 4.1.NettyRemotingServer#registerProcessor

```java
public void registerProcessor(int requestCode, NettyRequestProcessor processor, ExecutorService executor) {
		Pair<NettyRequestProcessor, ExecutorService> pair = new Pair<>(processor, executorThis);
		this.processorTable.put(requestCode, pair);
}
```



#### 3.2.1.AllocateMappedFileService#mmapOperation

```java
AllocateRequest req = null;
try {
  //从优先级队列中获取文件创建请求,优先级顺序根据文件名的大小
  req = this.requestQueue.take();
  AllocateRequest expectedRequest = requestTable.get(req.getFilePath());
	//并发的情况下队列之中可能会有多个
  if (expectedRequest != req) {
    return true;
  }
	if (req.getMappedFile() == null) {
    ...
    else {
      //初始化MappedFile
      mappedFile = new MappedFile(req.getFilePath(), req.getFileSize());
    }
  	req.setMappedFile(mappedFile);
    isSuccess = true;
  }
}finally {
	if (req != null && isSuccess)
    //下面的源码分析中会有阻塞等待这里创建MappedFile的逻辑
    req.getCountDownLatch().countDown();
}
```

#### 

