**核心概念**

```java
WorkQueue
```

---

**1.lockRunState**

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



**2.externalSubmit**

```java

```

