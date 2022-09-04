**属性**

```java
//线程池的状态和线程池中的线程数量都在这个变量里面
AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

int COUNT_BITS  = Integer.SIZE - 3;      //29

//线程数量
int CAPACITY    = (1 << COUNT_BITS) - 1; //0001 1111 1111 1111 1111 1111 1111 1111

int RUNNING     = -1 << COUNT_BITS       //1110 0000 0000 0000 0000 0000 0000 0000
int SHUTDOWN    = 0 << COUNT_BITS        //0000 0000 0000 0000 0000 0000 0000 0000
int STOP        = 1 << COUNT_BITS        //0010 0000 0000 0000 0000 0000 0000 0000
```

**位运算方法**

```java
//计算线程池的状态
private static int runStateOf(int c)     { return c & ~CAPACITY; }
//计算线程池中的线程数量
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
```
