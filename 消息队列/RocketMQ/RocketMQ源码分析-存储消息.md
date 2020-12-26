#### 相关类的初始化

1.进入BrokerStartup#main初始化Broker过程

```java
BrokerController brokerController = createBrokerController(args);
↓
↓
final BrokerController controller = new BrokerController(brokerConfig, nettyServerConfig, nettyClientConfig, messageStoreConfig);
boolean initResult = controller.initialize();
↓
↓
this.messageStore = new DefaultMessageStore(this.messageStoreConfig, this.brokerStatsManager, this.messageArrivingListener, this.brokerConfig); //2
...
//Netty
this.remotingServer = new NettyRemotingServer(this.nettyServerConfig, this.clientHousekeepingService);
this.registerProcessor(); //3
```

2.进入DefaultMessageStore构造方法

```java
this.commitLog = new CommitLog(this);
↓
↓
this.mappedFileQueue = new MappedFileQueue(defaultMessageStore.getMessageStoreConfig().getStorePathCommitLog(), defaultMessageStore.getMessageStoreConfig().getMappedFileSizeCommitLog(), defaultMessageStore.getAllocateMappedFileService());
↓
↓
//mappedFiles映射底层commitlog中的文件
private final CopyOnWriteArrayList<MappedFile> mappedFiles = new CopyOnWriteArrayList<MappedFile>();
```

3.进入BrokerController#registerProcessor方法

```java
SendMessageProcessor sendProcessor = new SendMessageProcessor(this);
this.remotingServer.registerProcessor(RequestCode.SEND_MESSAGE_V2, sendProcessor, this.sendMessageExecutor);
//源码中可以看到fastRemotingServer不处理PULL_MESSAGE类型的请求
//所以使用fastRemotingServer可以更快的处理其他类型的请求
this.fastRemotingServer.registerProcessor(RequestCode.SEND_MESSAGE_V2, sendProcessor, this.sendMessageExecutor);
↓
↓
//进入registerProcessor方法
Pair<NettyRequestProcessor, ExecutorService> pair = new Pair<NettyRequestProcessor, ExecutorService>(processor, executorThis);
this.processorTable.put(requestCode, pair);
```

---

### 消息存储请求的处理

1.进入remotingServer的实现类NettyRemotingServer的父类NettyRemotingAbstract#processMessageReceived

```java
case REQUEST_COMMAND:
    processRequestCommand(ctx, cmd);
↓
↓
//获取上一步创建的Pair
final Pair<NettyRequestProcessor, ExecutorService> matched = this.processorTable.get(cmd.getCode());
...
if (pair.getObject1() instanceof AsyncNettyRequestProcessor) {
    AsyncNettyRequestProcessor processor = (AsyncNettyRequestProcessor)pair.getObject1();
    processor.asyncProcessRequest(ctx, cmd, callback);
}
↓
↓
//AsyncNettyRequestProcessor#asyncProcessRequest
RemotingCommand response = processRequest(ctx, request);
↓
↓
//SendMessageProcessor#processRequest
response = asyncProcessRequest(ctx, request).get();
↓
↓
default:
	else{
    return this.asyncSendMessage(ctx, request, mqtraceContext, requestHeader);
  }
↓
↓
//构建消息
MessageExtBrokerInner msgInner = new MessageExtBrokerInner();
msgInner.setTopic(requestHeader.getTopic());
msgInner.setQueueId(queueIdInt);
...
else {
		putMessageResult = this.brokerController.getMessageStore().asyncPutMessage(msgInner);
}
↓
↓
CompletableFuture<PutMessageResult> putResultFuture = this.commitLog.asyncPutMessage(msg);
↓
↓
//默认情况下自旋锁
//使用AtomicBoolean实现
putMessageLock.lock();
try{
  if (null == mappedFile || mappedFile.isFull()) {
    //获取MappedFile
    mappedFile = this.mappedFileQueue.getLastMappedFile(0);
	}
	result = mappedFile.appendMessage(msg, this.appendMessageCallback);
} finally {
  //释放锁
  putMessageLock.unlock();
}
↓
↓
return appendMessagesInner(msg, cb);
↓
↓
int currentPos = this.wrotePosition.get();
if (currentPos < this.fileSize) {
  ByteBuffer byteBuffer = writeBuffer != null ? writeBuffer.slice() : this.mappedByteBuffer.slice();
  byteBuffer.position(currentPos);
  if (messageExt instanceof MessageExtBrokerInner) {
    result = cb.doAppend(this.getFileFromOffset(), byteBuffer, this.fileSize - currentPos, (MessageExtBrokerInner) messageExt);
	}
}
↓
↓
//构建写入MappedByteBuffer的内容
this.msgStoreItemMemory.putInt(msgLen);
this.msgStoreItemMemory.putInt(CommitLog.MESSAGE_MAGIC_CODE);
...
//写入MappedByteBuffer
byteBuffer.put(this.msgStoreItemMemory.array(), 0, msgLen);
//生成返回值
AppendMessageResult result = new AppendMessageResult(AppendMessageStatus.PUT_OK, wroteOffset, msgLen, msgId, msgInner.getStoreTimestamp(), queueOffset, CommitLog.this.defaultMessageStore.now() - beginTimeMills);
```

