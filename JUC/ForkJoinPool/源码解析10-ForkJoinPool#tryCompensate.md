**tryCompensate(WorkQueue w)**

```java
boolean canBlock;
WorkQueue[] ws; long c; int m, pc, sp;
if (w == null || w.qlock < 0 ||
    (ws = workQueues) == null || (m = ws.length - 1) <= 0 ||
    //获取parallelism
    (pc = config & SMASK) == 0)
    canBlock = false;
//判断是否存在空闲线程
else if ((sp = (int)(c = ctl)) != 0)
  	//唤醒空闲线程,补偿成功,返回true
    canBlock = tryRelease(c, ws[sp & m], 0L);
else {
  	//AC和TC
    int ac = (int)(c >> AC_SHIFT) + pc;
  	int tc = (short)(c >> TC_SHIFT) + pc;
    int nbusy = 0;
  	//遍历两次WorkQueues数组奇数位的WorkQueue
    for (int i = 0; i <= m; ++i) {
        WorkQueue v;
        if ((v = ws[((i << 1) | 1) & m]) != null) {
          	//如果WorkQueues数组存在扫描中的WorkQueue就停止遍历
            if ((v.scanState & SCANNING) != 0)
                break;
            ++nbusy;
        }
    }
  	//如果存在扫描中的WorkQueue,表示WorkQueues还不稳定,当前线程不能阻塞,返回false
    if (nbusy != (tc << 1) || ctl != c)
        canBlock = false;
  	//总线程数量达到7 && 活动线程数量大于1 && 工作队列中没有任务
  	//当前线程可以阻塞,但是这种阻塞属于没有补偿的阻塞,因为没有线程可以补偿了,返回true
    else if (tc >= pc && ac > 1 && w.isEmpty()) {
      	//AC - 1
        long nc = ((AC_MASK & (c - AC_UNIT)) | (~AC_MASK & c));
        canBlock = U.compareAndSwapLong(this, CTL, c, nc);
    }
  	...
    //补偿一个WorkQueue
  	else {
        boolean add = false; int rs;
      	//AC不变,TC + 1
        long nc = ((AC_MASK & c) | (TC_MASK & (c + TC_UNIT)));
        if (((rs = lockRunState()) & STOP) == 0)
            add = U.compareAndSwapLong(this, CTL, c, nc);
        unlockRunState(rs, rs & ~RSLOCK);
        canBlock = add && createWorker();
    }
}
return canBlock;
```

---

**canBlock = tryRelease(c, ws[sp & m], 0L);**

```java
为什么是0L?唤醒空闲线程,当前线程就会阻塞(awaitJoin方法中),AC总数量不变
```

