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

---

**lockRunState**

```java
private int lockRunState() {
	int rs = runState;
  //是否当前已经拿到锁了
  if((rs & RSLOCK) != 0){
    return awaitRunStateLock();
  }
  //没有拿到锁就上锁
  if(!U.compareAndSwapInt(this, RUNSTATE, rs, rs |= RSLOCK)){
    return awaitRunStateLock();
  }
  return rs;
}
```

**unLockRunState**

```java
private void unlockRunState(int oldRunState, int newRunState) {
  if (!U.compareAndSwapInt(this, RUNSTATE, oldRunState, newRunState)) {
  	Object lock = stealCounter;
    runState = newRunState;
    if (lock != null)
    	synchronized (lock) { lock.notifyAll(); }
    }
}
```

