**join()**

```java
int s;
if ((s = doJoin() & DONE_MASK) != NORMAL)
    reportException(s);
//返回任务结果
return getRawResult();
```

**doJoin()**

```java
int s; Thread t; ForkJoinWorkerThread wt; ForkJoinPool.WorkQueue w;
return (s = status) < 0 ? s :
            ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
            (w = (wt = (ForkJoinWorkerThread)t).workQueue).
            tryUnpush(this) && (s = doExec()) < 0 ? s :
            wt.pool.awaitJoin(w, this, 0L) :
            externalAwaitDone();
↓
↓
//我只关心这块逻辑,也是最难理解的部分
((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ? 
  (w = (wt = (ForkJoinWorkerThread)t).workQueue).tryUnpush(this) && (s = doExec()) < 0 ? 
  	s : wt.pool.awaitJoin(w, this, 0L)
↓
↓
//重写一下这块逻辑
if((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) {
  //尝试从工作队列头移除任务
  if((w = (wt = (ForkJoinWorkerThread)t).workQueue).tryUnpush(this)
		//执行任务
    && (s = doExec()) < 0) {
    return s;
  } else {
    return wt.pool.awaitJoin(w, this, 0L);
  }
}
↓
↓
//进入WorkQueue#tryUnpush
ForkJoinTask<?>[] a; int s;
if ((a = array) != null && (s = top) != base &&
    //当前任务如果在top位置,更新top位置为null,返回true
    U.compareAndSwapObject(a, (((a.length - 1) & --s) << ASHIFT) + ABASE, t, null)) {
  	//top - 1
    U.putOrderedInt(this, QTOP, s);
    return true;
}
return false;
```

---

