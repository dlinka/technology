**1.signalWork(WorkQueue[] ws, WorkQueue q)**

```java
long c; int sp, i; WorkQueue v; Thread p;
//AC初始化 = 1111111111111001
//CTL的前16位表示AC,AC最高位是1,CTL小于0,表示活动线程数不够
while ((c = ctl) < 0L) {
  	//CTL的后32位等于0表示当前没有空闲线程
  	//可能1个线程也没有,也可能1个空闲的线程都没有
    if ((sp = (int)c) == 0) {
        //TC初始化 = 1111111111111001
      	//CTL的33-48前位表示TC,TC最高位是1,表示总线程数不够
        if ((c & ADD_WORKER) != 0L)
            tryAddWorker(c); //2
        break;
    }
  	if(ws == null)
      	break;
  	//i等于sp的后16位
    if (ws.length <= (i = sp & SMASK))
        break;
    if ((v = ws[i]) == null)
        break;
  	//计算等待线程栈栈顶WorkQueue激活后新的scanState值
  	//增加1次值变化(SS_SEQ)和修改WorkQueue的扫描状态
    int vs = (sp + SS_SEQ) & ~INACTIVE;
  	//判断等待线程栈栈顶WorkQueue的scanState有没有发生过变化(这个过程可能存在ABA,具体看scan方法解析)
    int d = sp - v.scanState;
  	//CTL活动线程数+1
  	//CTL等待线程栈栈顶索引变更为下一个
    long nc = (UC_MASK & (c + AC_UNIT)) | (SP_MASK & v.stackPred);
    if (d == 0 && U.compareAndSwapLong(this, CTL, c, nc)) {
        v.scanState = vs;
        if ((p = v.parker) != null)
            U.unpark(p);
        break;
    }
 	  ...
}
```

**2.tryAddWorker(long c)**

```java
boolean add = false;
do {
  	//AC + 1
  	//TC + 1
    long nc = ((AC_MASK & (c + AC_UNIT)) | (TC_MASK & (c + TC_UNIT)));
    if (ctl == c) {
        int rs, stop;
        if ((stop = (rs = lockRunState()) & STOP) == 0)
            //CTL活动线程数+1
          	//CTL总线程数+1
          	add = U.compareAndSwapLong(this, CTL, c, nc);
        unlockRunState(rs, rs & ~RSLOCK);
      	...
        if (add) {
            createWorker(); //3
            break;
        }
    }
} while (((c = ctl) & ADD_WORKER) != 0L && (int)c == 0);
```

**3.createWorker()**

```java
ForkJoinWorkerThreadFactory fac = factory;
Throwable ex = null;
ForkJoinWorkerThread wt = null;
try {
  	//fac的默认实现类为CommonPoolForkJoinWorkerThreadFactory
  	//创建线程
    if (fac != null && (wt = fac.newThread(this)) != null) { //4
        //启动线程
      	wt.start(); //5
        return true;
    }
} catch (Throwable rex) { ... }
...
return false;
```

**4.registerWorker(ForkJoinWorkerThread wt)**

```java
return new ForkJoinWorkerThread(pool, null);
↓
↓
this.pool = pool;
this.workQueue = pool.registerWorker(this);
↓
↓
...
//将线程与WorkQueue绑定
WorkQueue w = new WorkQueue(this, wt);
int i = 0;
//默认值 = LIFO_QUEUE
int mode = config & MODE_MASK;
int rs = lockRunState();
try {
    WorkQueue[] ws; int n;
    if ((ws = workQueues) != null && (n = ws.length) > 0) {
      	//第一次 = 0x9e3779b9
      	int s = indexSeed += SEED_INCREMENT;
      	//n = 10000
      	//m = 01111
      	int m = n - 1;
      	//计算一个奇数数组下标
        i = ((s << 1) | 1) & m;
      	//不考虑这种情况
      	if(ws[i] != null) { ... }
        w.hint = s;
        //奇数数组索引｜任务出队模式
        w.config = i | mode;
        w.scanState = i;
        ws[i] = w;
    }
} finally {
    unlockRunState(rs, rs & ~RSLOCK);
}
//设置线程名
wt.setName(workerNamePrefix.concat(Integer.toString(i >>> 1)));
return w;
```

**5.runWorker(WorkQueue w)**

```java
//ForkJoinWorkerThread#run()
pool.runWorker(workQueue);
↓
↓
//第一次执行growArray方法会得到一个长度为8192的任务数组
w.growArray();
int seed = w.hint;
//第一次r = 0x9e3779b9
int r = (seed == 0) ? 1 : seed;
for (ForkJoinTask<?> t;;) {
  	//扫描任务
    if ((t = scan(w, r)) != null)
      	//执行任务
        w.runTask(t);
    else if (!awaitWork(w, r))
        break;
  	//一种伪随机算法(Xorshift)
    r ^= r << 13; r ^= r >>> 17; r ^= r << 5;
}
```

---

**为什么s << 1(i = ((s << 1) | 1) & m)?**

```java
//左移1位i值变化
3
5
7
9
11
13
15
1
  
//不左移1位i值变化
9
3
11
5
13
7
15
9
```

