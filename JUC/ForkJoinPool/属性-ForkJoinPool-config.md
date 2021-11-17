### config

**int**

| **位**    | **描述**                         |
| --------- | -------------------------------- |
| **32-17** | **mode = config & MODE_MASK**    |
| **16-1**  | **parallelism = config & SMARK** |

| 常量                         | **二进制**                            |
| ---------------------------- | ------------------------------------- |
| **SMASK = 0xffff**           | **0000000000000000 1111111111111111** |
| **MODE_MASK = 0xffff << 16** | **1111111111111111 0000000000000000** |
| **LIFO_QUEUE = 0**           | **0000000000000000 0000000000000000** |
| **FIFO_QUEUE = 1 << 16**     | **0000000000000001 0000000000000000** |
| **SHARED_QUEUE = 1 << 31**   | **1000000000000000 0000000000000000** |

---

**ForkJoinPool**

```java
private ForkJoinPool(int parallelism,
                         ForkJoinWorkerThreadFactory factory,
                         UncaughtExceptionHandler handler,
                         int mode,
                         String workerNamePrefix) {
  ...
  //我的CPU是8核,parallelism等于7
	//mode默认等于LIFO_QUEUE
	this.config = (parallelism & SMASK) | mode;
}
↓
↓
(parallelism & SMASK) //这行是保证parallelism不能大于SMASK
  0000000000000000 0000000000000111
  0000000000000000 1111111111111111
| mode
  0000000000000000 0000000000000111
  0000000000000000 0000000000000000
↓
↓
0000000000000000 0000000000000111
```

**通过config获取mode**

```java
config & MODE_MASK
  0000000000000000 0000000000000111
  1111111111111111 0000000000000000
↓
↓
0000000000000000 0000000000000000 //LIFO_QUEUE
```

---



