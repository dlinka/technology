**config**

```java
parallelism使用1-16位表示
mode使用17-32位表示
```

---

**ForkJoinPool构造函数**

```java
this.config = (parallelism & SMASK) | mode;
↓
↓
//parallelism等于availableProcessors-1,我电脑是8核,所以parallelism等于7
//这行是保证parallelism不能大于SMASK
parallelism & SMASK
  0000000000000000 0000000000000111
  0000000000000000 1111111111111111
//mode默认等于LIFO_QUEUE
| mode
  0000000000000000 0000000000000111
  0000000000000000 0000000000000000
↓
↓
0000000000000000 0000000000000111
```

**如何通过config获取parallelism**

```java
config & SMARK
```

**如何通过config获取mode**

```java
config & MODE_MASK
  0000000000000000 0000000000000111
  1111111111111111 0000000000000000
↓
↓
//LIFO_QUEUE
0000000000000000 0000000000000000
```

---

**二进制**

```java
//0000000000000000 1111111111111111
static final int SMASK        = 0xffff;

//1111111111111111 0000000000000000
static final int MODE_MASK    = 0xffff << 16;
//0000000000000000 0000000000000000
static final int LIFO_QUEUE   = 0;
//0000000000000001 0000000000000000
static final int FIFO_QUEUE   = 1 << 16;
```

