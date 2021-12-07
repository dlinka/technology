#### 1.BrokerStartup#main

```java
public static void main(String[] args) {
  start(createBrokerController(args));
}
↓
↓
public static BrokerController createBrokerController(String[] args) {
  final BrokerController controller = new BrokerController(
    brokerConfig, nettyServerConfig, nettyClientConfig, messageStoreConfig);
  controller.initialize();
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

  //消息写入处理的线程池
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

  //用来创建CommitLog的线程启动
  this.allocateMappedFileService.start(); //3.2
}
```

#### 3.1.CommitLog的构造方法

```java
public CommitLog(final DefaultMessageStore defaultMessageStore) {
  this.mappedFileQueue = new MappedFileQueue(
    //~/store/commitlog
    defaultMessageStore.getMessageStoreConfig().getStorePathCommitLog(),
    //1G = 1024 * 1024 *1024
    defaultMessageStore.getMessageStoreConfig().getMappedFileSizeCommitLog(),
    defaultMessageStore.getAllocateMappedFileService());
  
  //并发写入消息用到的锁
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
private boolean mmapOperation() {
  boolean isSuccess = false;
	AllocateRequest req = null;
  try {
    //从优先级阻塞队列中获取CommitLog的创建请求
    //优先级顺序根据文件名
		req = this.requestQueue.take();
    if (req.getMappedFile() == null) {
      //创建CommitLog
    	MappedFile mappedFile = new MappedFile(req.getFilePath(), req.getFileSize());
      req.setMappedFile(mappedFile);
      isSuccess = true;
    }
  } finally {
    if (req != null && isSuccess)
      //创建完成,"门栓"打开
    	req.getCountDownLatch().countDown();
  }
  return true;
}
↓
↓
public MappedFile(final String fileName, final int fileSize) throws IOException {
  init(fileName, fileSize);
}
↓
↓
private void init(final String fileName, final int fileSize) throws IOException {
	this.fileName = fileName;
  this.fileSize = fileSize;
  this.file = new File(fileName);
  this.fileFromOffset = Long.parseLong(this.file.getName());
  
  //MMAP
  this.fileChannel = new RandomAccessFile(this.file, "rw").getChannel();
  this.mappedByteBuffer = this.fileChannel.map(MapMode.READ_WRITE, 0, fileSize);
}
```

#### 4.BrokerController#registerProcessor

```java
public void registerProcessor() {
  //消息写入处理类
  SendMessageProcessor sendProcessor = new SendMessageProcessor(this);
	//将消息写入处理类注册到NettyRemotingServer
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

---

