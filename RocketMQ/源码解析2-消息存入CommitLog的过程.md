#### 1.NettyServerHandler#channelRead0

```java
@Override
protected void channelRead0(ChannelHandlerContext ctx, RemotingCommand msg) throws Exception {
    processMessageReceived(ctx, msg);
}
↓
↓
//NettyRemotingAbstract#processMessageReceived
//NettyRemotingAbstract是NettyRemotingServer的父类
public void processMessageReceived(ChannelHandlerContext ctx, RemotingCommand msg) throws Exception {
		case REQUEST_COMMAND:
    	processRequestCommand(ctx, cmd);
}
↓
↓
//NettyRemotingAbstract#processRequestCommand
public void processRequestCommand(final ChannelHandlerContext ctx, final RemotingCommand cmd) {
  	//通过请求类型获取Pair
		final Pair<NettyRequestProcessor, ExecutorService> matched = this.processorTable.get(cmd.getCode());
		...
    Runnable run = new Runnable() {
    		@Override
        public void run() {
        		if (pair.getObject1() instanceof AsyncNettyRequestProcessor) {
            		AsyncNettyRequestProcessor processor = (AsyncNettyRequestProcessor)pair.getObject1();
            		processor.asyncProcessRequest(ctx, cmd, callback); //2
        		}
        }
    }
  	...
    final RequestTask requestTask = new RequestTask(run, ctx.channel(), cmd);
  	//提交到线程池去执行
    pair.getObject2().submit(requestTask);
}
```

#### 2.SendMessageProcessor#asyncProcessRequest

```java
public void asyncProcessRequest(ChannelHandlerContext ctx, RemotingCommand request, RemotingResponseCallback responseCallback) throws Exception {
  	asyncProcessRequest(ctx, request).thenAcceptAsync(responseCallback::callback, brokerController.getSendMessageExecutor());
}
↓
↓
public CompletableFuture<RemotingCommand> asyncProcessRequest(ChannelHandlerContext ctx,
                                                                  RemotingCommand request) throws RemotingCommandException {
  	return asyncSendMessage(ctx, request, mqtraceContext, requestHeader);
}
↓
↓
private CompletableFuture<RemotingCommand> asyncSendMessage(ChannelHandlerContext ctx, RemotingCommand request, SendMessageContext mqtraceContext, SendMessageRequestHeader requestHeader) {
		//MessageExtBrokerInner
		MessageExtBrokerInner msgInner = new MessageExtBrokerInner();
		msgInner.setTopic(requestHeader.getTopic());
		msgInner.setQueueId(queueIdInt);
		...
		else {
				putMessageResult = this.brokerController.getMessageStore().asyncPutMessage(msgInner);
		}
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
		//之前使用的是自旋锁,现在默认情况使用ReentrantLock
    putMessageLock.lock();
    try{
				MappedFile mappedFile = this.mappedFileQueue.getLastMappedFile();
      	if (null == mappedFile || mappedFile.isFull()) {
        	//创建CommitLog文件
        	mappedFile = this.mappedFileQueue.getLastMappedFile(0); //5
      	}
      result = mappedFile.appendMessage(msg, this.appendMessageCallback); //6
    } finally {
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
  	MappedFile mappedFileLast = getLastMappedFile();
  	if (mappedFileLast == null) {
      	//第一次等于0
    		createOffset = startOffset - (startOffset % this.mappedFileSize);
		}
  	if (createOffset != -1 && needCreate) {
      	//一次创建两个
      	//路径等于/Users/dlinka/store/commitlog/00000000000000000000
    		String nextFilePath = this.storePath + File.separator + UtilAll.offset2FileName(createOffset);
      	//路径等于/Users/dlinka/store/commitlog/00000000001073741824
      	String nextNextFilePath = this.storePath + File.separator + UtilAll.offset2FileName(createOffset + this.mappedFileSize);
      	MappedFile mappedFile = null;
      	if (this.allocateMappedFileService != null) {
    				mappedFile = this.allocateMappedFileService.putRequestAndReturnMappedFile(nextFilePath, nextNextFilePath, this.mappedFileSize); //5.1
				}
      	...
        if (mappedFile != null) {
    				if (this.mappedFiles.isEmpty()) {
        				mappedFile.setFirstCreateInQueue(true);
    				}
          	//加入List
    				this.mappedFiles.add(mappedFile);
				}
      	return mappedFile;
    }
}

```

#### 5.1.AllocateMappedFileService#putRequestAndReturnMappedFile

```java
int canSubmitRequests = 2;
//第一个MappedFile创建请求
AllocateRequest nextReq = new AllocateRequest(nextFilePath, fileSize);
boolean nextPutOK = requestTable.putIfAbsent(nextFilePath, nextReq) == null;
if (nextPutOK) {
  //添加到优先级队列
	boolean offerOK = requestQueue.offer(nextReq);
}
//创建第二个请求
AllocateRequest nextNextReq = new AllocateRequest(nextNextFilePath, fileSize);
boolean nextNextPutOK = requestTable.putIfAbsent(nextNextFilePath, nextNextReq) == null;
if (nextNextPutOK) {
	//添加到优先级队列
  boolean offerOK = requestQueue.offer(nextNextReq);
}
...
//只拿出第一个请求
AllocateRequest result = requestTable.get(nextFilePath);
if (result != null) {
  //阻塞等待上面创建MappedFile
	result.getCountDownLatch().await(waitTimeOut, TimeUnit.MILLISECONDS);
  requestTable.remove(nextFilePath);
	return result.getMappedFile();
}
```











5.MappedFile#appendMessage

```java
return appendMessagesInner(msg, cb);
↓
↓
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

