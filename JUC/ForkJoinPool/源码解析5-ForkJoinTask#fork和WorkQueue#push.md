**ForkJoinTask#fork()**

```java
Thread t;
	if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
		//调用fork的任务push到工作队列的队首
		((ForkJoinWorkerThread)t).workQueue.push(this);
	else
	  ...
return this;
```

**WorkQueue#push(ForkJoinTask<?> task)**

```java
ForkJoinTask<?>[] a; ForkJoinPool p;
int b = base, s = top, n;
if ((a = array) != null) {
	int m = a.length - 1;
	//任务放到WorkQueue的top的位置
	U.putOrderedObject(a, ((m & s) << ASHIFT) + ABASE, task);
	//top + 1
	U.putOrderedInt(this, QTOP, s + 1);
	//如果队列没有任务(等于0),那么很有可能就是任务内首次执行fork方法,这时可以尝试唤醒等待线程栈栈顶WorkQueue帮助执行任务,提升任务执行效率
	//还有其他几种情况,其实目的都是一个,提升整体执行效率
	if ((n = s - b) <= 1) {
		if ((p = pool) != null)
			p.signalWork(p.workQueues, this);
	}
	...
}
```

---
