#### 1.NettyRemotingAbstract#processMessageReceived

**NettyRemotingAbstract是NettyRemotingServer的父类**

```java
public void processMessageReceived(ChannelHandlerContext ctx, RemotingCommand msg) throws Exception {
  switch (cmd.getType()) {
    case REQUEST_COMMAND:
      processRequestCommand(ctx, cmd);
  }
}
↓
↓
public void processRequestCommand(final ChannelHandlerContext ctx, final RemotingCommand cmd) {
  final Pair<NettyRequestProcessor, ExecutorService> matched = this.processorTable.get(cmd.getCode());
	//线程池运行的Runnable
  Runnable run = new Runnable() {
    public void run() {
      if (pair.getObject1() instanceof AsyncNettyRequestProcessor) {
        AsyncNettyRequestProcessor processor = (AsyncNettyRequestProcessor)pair.getObject1();
        processor.asyncProcessRequest(ctx, cmd, callback);
      }
    }
  }

  final RequestTask requestTask = new RequestTask(run, ctx.channel(), cmd);
  //提交线程池处理任务
  pair.getObject2().submit(requestTask);
}
```

#### 2.SendMessageProcessor#asyncProcessRequest

```java
public void asyncProcessRequest(ChannelHandlerContext ctx, RemotingCommand request, RemotingResponseCallback responseCallback) throws Exception {
  asyncProcessRequest(ctx, request);
}
↓
↓
public CompletableFuture<RemotingCommand> asyncProcessRequest(ChannelHandlerContext ctx, RemotingCommand request) throws RemotingCommandException {
  return this.asyncSendMessage(ctx, request, mqtraceContext, requestHeader);
}
↓
↓
private CompletableFuture<RemotingCommand> asyncSendMessage(ChannelHandlerContext ctx, RemotingCommand request, SendMessageContext mqtraceContext, SendMessageRequestHeader requestHeader) {
  //创建MessageExtBrokerInner
  MessageExtBrokerInner msgInner = new MessageExtBrokerInner();
	//设置topic
  msgInner.setTopic(requestHeader.getTopic());

  putMessageResult = this.brokerController.getMessageStore().asyncPutMessage(msgInner);
}
```

#### 3.DefaultMessageStore#asyncPutMessage

```java
public CompletableFuture<PutMessageResult> asyncPutMessage(MessageExtBrokerInner msg) {
  CompletableFuture<PutMessageResult> putResultFuture = this.commitLog.asyncPutMessage(msg);
}
```

#### 4.CommitLog#asyncPutMessage

```java
public CompletableFuture<PutMessageResult> asyncPutMessage(final MessageExtBrokerInner msg) {
  //写入结果
  AppendMessageResult result = null;
  //获取最后一个CommitLog
  MappedFile mappedFile = this.mappedFileQueue.getLastMappedFile();
  //低版本这里使用的自旋锁,高并发非常影响机器性能,这里建议改成ReentrantLock
  putMessageLock.lock();
  try{
    if (null == mappedFile || mappedFile.isFull()) {
      //创建CommitLog文件
      mappedFile = this.mappedFileQueue.getLastMappedFile(0); //5
    }
    //写入消息
    result = mappedFile.appendMessage(msg, this.appendMessageCallback); //6
  } finally {
    //释放锁
    putMessageLock.unlock();
  }
}
```

#### 5.MappedFileQueue#getLastMappedFile

```java
public MappedFile getLastMappedFile(final long startOffset) {
  return getLastMappedFile(startOffset, true);
}
↓
↓
public MappedFile getLastMappedFile(final long startOffset, boolean needCreate) {
  long createOffset = -1;
  MappedFile mappedFileLast = getLastMappedFile();
  if (mappedFileLast == null) {
    createOffset = startOffset - (startOffset % this.mappedFileSize);
  }
  if (createOffset != -1 && needCreate) {
    //path=/Users/dlinka/store/commitlog/00000000000000000000
    String nextFilePath = this.storePath + File.separator + UtilAll.offset2FileName(createOffset);
    //path=/Users/dlinka/store/commitlog/00000000001073741824
    String nextNextFilePath = this.storePath + File.separator + UtilAll.offset2FileName(createOffset + this.mappedFileSize);
      
    MappedFile mappedFile = null;
    if (this.allocateMappedFileService != null) {
      //真正的创建CommitLog
      mappedFile = this.allocateMappedFileService.putRequestAndReturnMappedFile(nextFilePath, nextNextFilePath, this.mappedFileSize); //5.1
    }
    if (mappedFile != null) {
      this.mappedFiles.add(mappedFile);
    }
    return mappedFile;
  }
}
```

#### 5.1.AllocateMappedFileService#putRequestAndReturnMappedFile

```java
public MappedFile putRequestAndReturnMappedFile(String nextFilePath, String nextNextFilePath, int fileSize) {
  //第一个CommitLog创建请求
  AllocateRequest nextReq = new AllocateRequest(nextFilePath, fileSize);
  boolean nextPutOK = this.requestTable.putIfAbsent(nextFilePath, nextReq) == null;
  if (nextPutOK) {
  	//将第一个AllocateRequest添加到优先级队列
		this.requestQueue.offer(nextReq);
  }
  
  //第二个CommitLog创建请求
  AllocateRequest nextNextReq = new AllocateRequest(nextNextFilePath, fileSize);
  boolean nextNextPutOK = this.requestTable.putIfAbsent(nextNextFilePath, nextNextReq) == null;
  if (nextNextPutOK) {
    //将第二个AllocateRequest添加到优先级队列
  	this.requestQueue.offer(nextNextReq);
  }
  
  //拿出第一个AllocateRequest请求
  AllocateRequest result = this.requestTable.get(nextFilePath);
  if (result != null) {
    //阻塞等待CommitLog的创建,等待"门栓"打开
  	boolean waitOK = result.getCountDownLatch().await(waitTimeOut, TimeUnit.MILLISECONDS);
    if (!waitOK) { //超时创建
      return null;
    } else {
      this.requestTable.remove(nextFilePath);
      //返回创建成功的CommitLog
      return result.getMappedFile();
    }
  }
  return null;
}
```

#### 6.MappedFile#appendMessage

```java
public AppendMessageResult appendMessage(final MessageExtBrokerInner msg, final AppendMessageCallback cb) {
  return appendMessagesInner(msg, cb);
}
↓
↓
public AppendMessageResult appendMessagesInner(final MessageExt messageExt, final AppendMessageCallback cb) {
  //当前写入的位置
	int currentPos = this.wrotePosition.get();
  //当前位置小于文件大小
  if (currentPos < this.fileSize) {
    ByteBuffer byteBuffer = writeBuffer != null ? writeBuffer.slice() : this.mappedByteBuffer.slice();
  }
}
```









5.MappedFile#appendMessage

```java

ByteBuffer byteBuffer = writeBuffer != null ? writeBuffer.slice() : mappedByteBuffer.slice();
byteBuffer.position(currentPos);
if (messageExt instanceof MessageExtBrokerInner) {
	result = cb.doAppend(getFileFromOffset(), byteBuffer, fileSize - currentPos, (MessageExtBrokerInner) messageExt);
}
```

6.DefaultAppendMessageCallback#doAppend

```java
this.msgStoreItemMemory.putInt(msgLen);
this.msgStoreItemMemory.putInt(CommitLog.MESSAGE_MAGIC_CODE);
...
byteBuffer.put(this.msgStoreItemMemory.array(), 0, msgLen);
...
AppendMessageResult result = new AppendMessageResult(AppendMessageStatus.PUT_OK, wroteOffset, msgLen, msgId, msgInner.getStoreTimestamp(), queueOffset, CommitLog.this.defaultMessageStore.now() - beginTimeMills);
...
switch (tranType) {
	case MessageSysFlag.TRANSACTION_NOT_TYPE:
  //默认情况tranType等于0,所以这里也会被执行
  //这里主要用来设置QUEUEOFFSET
	case MessageSysFlag.TRANSACTION_COMMIT_TYPE:
		CommitLog.this.topicQueueTable.put(key, ++queueOffset);
}
```

---

