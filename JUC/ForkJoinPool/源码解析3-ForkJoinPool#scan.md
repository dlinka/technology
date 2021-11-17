**scan(WorkQueue w, int r)**

```java
WorkQueue[] ws; int m;
if ((ws = workQueues) != null && (m = ws.length - 1) > 0 && w != null) {	
    //初始化等于奇数数组索引
  	int ss = w.scanState;
    //origin表示循环WorkQueue数组的第一次的索引位置
  	for (int origin = r & m, k = origin, oldSum = 0, checkSum = 0;;) {
        WorkQueue q; ForkJoinTask<?>[] a; ForkJoinTask<?> t;
        int b, n; long c;
        if ((q = ws[k]) != null) {
          	//检查这个WorkQueue是否有可以窃取的任务
            if ((n = (b = q.base) - q.top) < 0 && (a = q.array) != null) {
              	long i = (((a.length - 1) & b) << ASHIFT) + ABASE;
              	//获取base位置任务 && 这个过程base没有发生过变化
              	if ((t = ((ForkJoinTask<?>) U.getObjectVolatile(a, i))) != null && q.base == b) {
                  	//scanState大于0表示这个WorkQueue为扫描状态
                  	if (ss >= 0) {
                        if (U.compareAndSwapObject(a, i, t, null)) {
                            q.base = b + 1;
                            if (n < -1)
                                signalWork(ws, q);
                            return t;
                        }
                    }
                  	//不懂这里为什么要判断oldSum == 0? 有懂的解释下
                    //为什么要判断scanState < 0? 其他WorkQueue这时可能会唤醒栈顶空闲WorkQueue,也就是当前WorkQueue
                    else if (oldSum == 0 && w.scanState < 0)
                        tryRelease(c = ctl, ws[m & (int)c], AC_UNIT);
                }
              	//其他WorkQueue这时可能会唤醒栈顶空闲WorkQueue,也就是当前WorkQueue
              	//不过也可能是自己上面调用tryRelease唤醒
                if (ss < 0)
                    ss = w.scanState;
              	//重新找个随机数,全部重新开始
                r ^= r << 1; r ^= r >>> 3; r ^= r << 10;
                origin = k = r & m;
                oldSum = checkSum = 0;
                continue;
            }
          	//叠加base值
            checkSum += b;
        }
      	//线性查找下一个位置判断是否回到第一次的索引位置
        if ((k = (k + 1) & m) == origin) {
          	//为什么要重新赋值scanState? 其他WorkQueue这时可能会唤醒栈顶空闲WorkQueue,也就是当前WorkQueue
            if ((ss >= 0 || (ss == (ss = w.scanState))) &&
                //利用oldSum和checkSum进行两次扫描,检查两次扫描base总和是否一致,如果一致当前线程就可以尝试挂起
                oldSum == (oldSum = checkSum)) {
                //死循环出口
              	if (ss < 0 || w.qlock < 0)
                    break;
                int ns = ss | INACTIVE;
                long nc = ((SP_MASK & ns) | (UC_MASK & ((c = ctl) - AC_UNIT)));
								//当前WorkQueue保存前一个空闲WorkQueue
              	w.stackPred = (int)c;
                U.putInt(w, QSCANSTATE, ns);
                if (U.compareAndSwapLong(this, CTL, c, nc))
                    ss = ns;
                else
                    w.scanState = ss;
            }
            checkSum = 0;
        }
    }
}
return null;
```

---

