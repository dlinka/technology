**核心概念**

```java
WorkQueue
```

---

**externalSubmit**

```java
private void externalSubmit(ForkJoinTask<?> task) {
	for(;;) {
    WorkQueue[] ws; WorkQueue q; int rs, m, k;
    boolean move = false;
    
    //初始化
    else if((rs & STARTED) == 0 ||
           (ws = workQueues) == null ||
           (m = ws.length - 1) < 0) {
      int ns = 0;
      //获取锁
      rs = lockRunState();
      try {
        if((rs & STARTED) == 0) {
          //初始化stealCounter属性
          U.compareAndSwapObject(this, STEALCOUNTER, null, new AtomicLong());
          //通过parallelism获取n的值
          //我的电脑是8核,所以n的值等于16
          int p = config & SMASK;
          int n = (p > 1) ? p - 1 : 1;
          n |= n >>> 1; n |= n >>> 2;  n |= n >>> 4; n |= n >>> 8; n |= n >>> 16;
          n = (n + 1) << 1;
          workQueues = new WorkQueue[n];
          ns = STARTED;
        }
      } finally {
        //解锁
      	unlockRunState(rs, (rs & ~RSLOCK) | ns);
      }
    }
    
    //r & m = 随机数 & (数组长度-1),得到一个数组索引
    //& SQMARK = & 1111110,得到一个最大值为126的偶数(为什么是偶数?SQMARSK最后一位是0)
    else if ((q = ws[k = r & m & SQMASK]) != null) {
      //对WorkQueue加锁
      if (q.qlock == 0 && U.compareAndSwapInt(q, QLOCK, 0, 1)) {
      	ForkJoinTask<?>[] a = q.array;
        int s = q.top;
        boolean submitted = false;
        try {
    			if ((a != null && a.length > s + 1 - q.base) || (a = q.growArray()) != null) {
						
            int j = (((a.length - 1) & s) << ASHIFT) + ABASE;
        		U.putOrderedObject(a, j, task);
        		U.putOrderedInt(q, QTOP, s + 1);
        		submitted = true;
    			}
				} finally {
    			U.compareAndSwapInt(q, QLOCK, 1, 0);
				}
        if (submitted) {
        	signalWork(ws, q);
          return;
        }
      }
    }
    
    else if(((rs = runState) & RSLOCK) == 0) {
      //初始化一个WorkQueue
      q = new WorkQueue(this, null);
      q.hint = r;
      q.config = k | SHARED_QUEUE;
      q.scanState = INACTIVE;
      
      //加锁
      rs = lockRunState(); 
      if (rs > 0 &&  (ws = workQueues) != null && k < ws.length && ws[k] == null) {
        ws[k] = q; //将任务赋值到WorkQueue数组的偶数位
      }
      //解锁
      unlockRunState(rs, rs & ~RSLOCK);
    }
  }
}
```

