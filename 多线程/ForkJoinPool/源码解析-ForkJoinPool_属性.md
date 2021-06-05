```java
//0000000000000000 1111111111111111
static final int SMASK        = 0xffff;
//0000000000000000 0111111111111111
static final int MAX_CAP      = 0x7fff;
//0000000000000000 1111111111111110
static final int EVENMASK     = 0xfffe;
//0000000000000000 0000000001111110
static final int SQMASK       = 0x007e;

//1111111111111111 0000000000000000
static final int MODE_MASK    = 0xffff << 16;
//0000000000000000 0000000000000000
static final int LIFO_QUEUE   = 0;
//0000000000000001 0000000000000000
static final int FIFO_QUEUE   = 1 << 16;
//1000000000000000 0000000000000000
static final int SHARED_QUEUE = 1 << 31;

```

**ctl**

```java
private static final int  AC_SHIFT   = 48;
//00000000000000001 0000000000000000 0000000000000000 0000000000000000
private static final long AC_UNIT    = 0x0001L << AC_SHIFT;
//01111111111111111 0000000000000000 0000000000000000 0000000000000000
private static final long AC_MASK    = 0xffffL << AC_SHIFT;

private static final int  TC_SHIFT   = 32;
//0000000000000000 0000000000000001 0000000000000000 0000000000000000
private static final long TC_UNIT    = 0x0001L << TC_SHIFT;
//0000000000000000 1111111111111111 0000000000000000 0000000000000000
private static final long TC_MASK    = 0xffffL << TC_SHIFT;
//0000000000000000 1000000000000000 0000000000000000 0000000000000000
private static final long ADD_WORKER = 0x0001L << (TC_SHIFT + 15);
```

**runState**

```java
private static final int  RSLOCK     = 1;
private static final int  RSIGNAL    = 1 << 1;
private static final int  STARTED    = 1 << 2;
private static final int  STOP       = 1 << 29;
//0100 0000 0000 0000 0000 0000 0000 0000
private static final int  TERMINATED = 1 << 30;
//1000 0000 0000 0000 0000 0000 0000 0000
private static final int  SHUTDOWN   = 1 << 31;
```

