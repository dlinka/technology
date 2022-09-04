#### 1.ThreadPoolExecutor#execute

```java
public void execute(Runnable command) {
  int c = ctl.get();
  if (workerCountOf(c) < corePoolSize) { //如果线程池中线程数量小于核心线程数
    if (addWorker(command, true))
      return; 
    c = ctl.get();
  }
  
  if (isRunning(c) && workQueue.offer(command)) { //如果线程数量大于核心线程数,任务放到阻塞队列中
  	...
  }
	else if (!addWorker(command, false)) { //如果阻塞队列满了,线程池中的线程数量小于最大线程数
    ...
  }
}
```

#### 2.ThreadPoolExecutor#addWorker

```java
private boolean addWorker(Runnable firstTask, boolean core) {
  retry:
  for (;;) {
    int c = ctl.get();
    for (;;) {
      int wc = workerCountOf(c); //当前线程数
      if (wc >= CAPACITY ||
          wc >= (core ? corePoolSize : maximumPoolSize)) //判断当前线程池中的线程数量是否大于核心线程数或者最大线程数
        return false;

      if (compareAndIncrementWorkerCount(c)) //更新线程池中线程数量+1
        break retry;
    }
  }
  
  boolean workerStarted = false;
  boolean workerAdded = false;
  Worker w = null;
  try {
    w = new Worker(firstTask); //Runnable对象当成Worker的第一个任务传入
    final Thread t = w.thread; //Worker的构造函数中会创建一个Thread
    if (t != null) {
      int rs = runStateOf(ctl.get());
      if (rs < SHUTDOWN || (rs == SHUTDOWN && firstTask == null)) {
				workers.add(w);
        workerAdded = true;
      }
      if (workerAdded) {
        t.start(); //线程执行
        workerStarted = true;
      } 
    }  
  }
  return workerStarted;
}
```

#### 3.Worker#run

```java
public void run() {
	runWorker(this);
}
↓
↓
//ThreadPoolExecutor#runWorker
final void runWorker(Worker w) {
	Runnable task = w.firstTask; //拿到第一个任务
  w.firstTask = null;
  try {
    while (task != null || (task = getTask()) != null) {	//第一次就拿firstTask去执行,之后从阻塞队列中拿任务执行
   		try {
      	task.run(); //任务执行
      } finally {
        task = null;
        w.completedTasks++;
      }
    }
  }
}

```

#### 4.ThreadPoolExecutor#getTask

```java
private Runnable getTask() {
	try {
    //从阻塞队列中获取任务
    Runnable r = timed ? workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) : workQueue.take();
    if (r != null)
      return r;
  }
}
```



---

