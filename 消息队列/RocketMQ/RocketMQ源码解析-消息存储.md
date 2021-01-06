##### 相关类的初始化

1.进入BrokerStartup#main

```java
BrokerController brokerController = createBrokerController(args);
↓
↓
final BrokerController controller = new BrokerController(brokerConfig, nettyServerConfig, nettyClientConfig, messageStoreConfig);
boolean initResult = controller.initialize();
↓
↓
//初始化DefaultMessageStore
this.messageStore = new DefaultMessageStore(this.messageStoreConfig, this.brokerStatsManager, this.messageArrivingListener, this.brokerConfig); //2
...
//初始化Netty
this.remotingServer = new NettyRemotingServer(this.nettyServerConfig, this.clientHousekeepingService);
...
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
//MappedFileQueue的mappedFiles属性
//MappedFile就是commitlog文件
private final CopyOnWriteArrayList<MappedFile> mappedFiles = new CopyOnWriteArrayList<MappedFile>();
```

3.进入BrokerController#registerProcessor

```java
//请求处理类
SendMessageProcessor sendProcessor = new SendMessageProcessor(this);
...
//把请求处理类祖注册到remotingServer中
this.remotingServer.registerProcessor(RequestCode.SEND_MESSAGE_V2, sendProcessor, this.sendMessageExecutor);
...
//把请求处理类祖注册到fastRemotingServer中
//fastRemotingServer不会处理PULL_MESSAGE类型的请求
//如果remotingServer处理不过来,可以利用fastRemotingServer处理相关类型请求
this.fastRemotingServer.registerProcessor(RequestCode.SEND_MESSAGE_V2, sendProcessor, this.sendMessageExecutor);
↓
↓
Pair<NettyRequestProcessor, ExecutorService> pair = new Pair<NettyRequestProcessor, ExecutorService>(processor, executorThis);
this.processorTable.put(requestCode, pair);
```

---

##### 消息存储

1.进入remotingServer的实现类NettyRemotingServer的父类NettyRemotingAbstract#processMessageReceived

```java
case REQUEST_COMMAND:
    processRequestCommand(ctx, cmd);
↓
↓
//获取Pair
final Pair<NettyRequestProcessor, ExecutorService> matched = this.processorTable.get(cmd.getCode());
...
if (pair.getObject1() instanceof AsyncNettyRequestProcessor) {
    AsyncNettyRequestProcessor processor = (AsyncNettyRequestProcessor)pair.getObject1();
    processor.asyncProcessRequest(ctx, cmd, callback);
}
```

2.SendMessageProcessor#asyncProcessRequest

```java
asyncProcessRequest(ctx, request).thenAcceptAsync(responseCallback::callback, this.brokerController.getSendMessageExecutor());
↓
↓
default:
	else{
    return this.asyncSendMessage(ctx, request, mqtraceContext, requestHeader);
  }
↓
↓
//构建MessageExtBrokerInner
MessageExtBrokerInner msgInner = new MessageExtBrokerInner();
msgInner.setTopic(requestHeader.getTopic());
msgInner.setQueueId(queueIdInt);
...
else {
		putMessageResult = this.brokerController.getMessageStore().asyncPutMessage(msgInner);
}
```

3.DefaultMessageStore#asyncPutMessage

```java
CompletableFuture<PutMessageResult> putResultFuture = this.commitLog.asyncPutMessage(msg);
```

4.CommitLog#asyncPutMessage

```java
//默认情况下使用自旋锁(AtomicBoolean)
putMessageLock.lock();
try{
  if (null == mappedFile || mappedFile.isFull()) {
    //获取MappedFile
    mappedFile = this.mappedFileQueue.getLastMappedFile(0);
	}
	result = mappedFile.appendMessage(msg, this.appendMessageCallback);
} finally {
  putMessageLock.unlock();
}
```

5.MappedFile#appendMessage

```java
return appendMessagesInner(msg, cb);
↓
↓
ByteBuffer byteBuffer = writeBuffer != null ? writeBuffer.slice() : this.mappedByteBuffer.slice();
byteBuffer.position(currentPos);
if (messageExt instanceof MessageExtBrokerInner) {
	result = cb.doAppend(this.getFileFromOffset(), byteBuffer, this.fileSize - currentPos, (MessageExtBrokerInner) messageExt);
}
```

6.DefaultAppendMessageCallback#doAppend

```java
this.msgStoreItemMemory.putInt(msgLen);
this.msgStoreItemMemory.putInt(CommitLog.MESSAGE_MAGIC_CODE);
...
byteBuffer.put(this.msgStoreItemMemory.array(), 0, msgLen);
...
//返回值
AppendMessageResult result = new AppendMessageResult(AppendMessageStatus.PUT_OK, wroteOffset, msgLen, msgId, msgInner.getStoreTimestamp(), queueOffset, CommitLog.this.defaultMessageStore.now() - beginTimeMills);
```

---

