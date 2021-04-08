**ctl:代表线程池的状态和线程数量**

```java
AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
↓
COUNT_BITS = Integer.SIZE - 3 = 29
CAPACITY = (1 << COUNT_BITS) - 1
  0001 1111 1111 1111 1111 1111 1111 1111
RUNNING = -1 << COUNT_BITS
  1110 0000 0000 0000 0000 0000 0000 0000
SHUTDOWN = 0 << COUNT_BITS
  0000 0000 0000 0000 0000 0000 0000 0000
STOP = 1 << COUNT_BITS
  0010 0000 0000 0000 0000 0000 0000 0000
```

**位运算方法**

ctlOf

```java
private static int ctlOf(int rs, int wc) { return rs | wc; }
↓
↓
ctlOf(RUNNING, 0);
↓
↓
1110 0000 0000 0000 0000 0000 0000 0000
 |(或运算)
0000 0000 0000 0000 0000 0000 0000 0000
↓
↓
1110 0000 0000 0000 0000 0000 0000 0000
```

runStateOf

```java
private static int runStateOf(int c) { return c & ~CAPACITY; }
↓
↓
~CAPACITY
  1110 0000 0000 0000 0000 0000 0000 0000
&
	两个位都为1时,才为1
↓
↓
计算前3位的值
```

workerCountOf

```java
private static int workerCountOf(int c)  { return c & CAPACITY; }
↓
↓
CAPACITY
  0001 1111 1111 1111 1111 1111 1111 1111
&
	两个位都为1时,才为1
↓
↓
计算后29位的值
```



