1.进入BrokerStartup#main

```java
BrokerController brokerController = createBrokerController(args);
↓
↓
final BrokerController controller = new BrokerController(brokerConfig, nettyServerConfig, nettyClientConfig, messageStoreConfig);
boolean initResult = controller.initialize();
↓
↓
this.messageStore = new DefaultMessageStore(this.messageStoreConfig, this.brokerStatsManager, this.messageArrivingListener, this.brokerConfig);
↓
//AllocateMappedFileService继承ServiceThread
this.allocateMappedFileService = new AllocateMappedFileService(this);
//线程开启
this.allocateMappedFileService.start();
```

2.进入AllocateMappedFileService#run

```java
while (!this.isStopped() && this.mmapOperation()) {}
↓
↓
//直接进入mmapOperation
//这里的逻辑是从优先级队列中获取文件创建请求,优先级顺序根据文件名的大小
try {
  req = this.requestQueue.take();
  AllocateRequest expectedRequest = this.requestTable.get(req.getFilePath());
	...
	if (req.getMappedFile() == null) {
		...
    else {
      //初始化MappedFile
      mappedFile = new MappedFile(req.getFilePath(), req.getFileSize());
    }
  	req.setMappedFile(mappedFile);
  }
}finally {
	if (req != null && isSuccess)
    //下面的源码分析中会有阻塞等待这里创建MappedFile的逻辑
    req.getCountDownLatch().countDown();
}
```

3.进入MappedFileQueue#getLastMappedFile

```java
if (this.allocateMappedFileService != null) {
  //利用AllocateMappedFileService线程异步创建MappedFile
  //同时创建两个MappedFile
	mappedFile = this.allocateMappedFileService.putRequestAndReturnMappedFile(nextFilePath, nextNextFilePath, this.mappedFileSize);
}
↓
↓
int canSubmitRequests = 2;
//创建第一个请求
AllocateRequest nextReq = new AllocateRequest(nextFilePath, fileSize);
boolean nextPutOK = this.requestTable.putIfAbsent(nextFilePath, nextReq) == null;
if (nextPutOK) {
  //添加到优先级队列
	boolean offerOK = this.requestQueue.offer(nextReq);
}
//创建第二个请求
AllocateRequest nextNextReq = new AllocateRequest(nextNextFilePath, fileSize);
boolean nextNextPutOK = this.requestTable.putIfAbsent(nextNextFilePath, nextNextReq) == null;
if (nextNextPutOK) {
	//添加到优先级队列
  boolean offerOK = this.requestQueue.offer(nextNextReq);
}
...
//只拿出第一个请求
AllocateRequest result = this.requestTable.get(nextFilePath);
if (result != null) {
  //阻塞等待上面创建MappedFile
	boolean waitOK = result.getCountDownLatch().await(waitTimeOut, TimeUnit.MILLISECONDS);
  else {
    this.requestTable.remove(nextFilePath);
  	return result.getMappedFile();
  }
}
```

