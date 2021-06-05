**核心概念**

```java
WorkQueue
```

---

**1.lockRunState**

```java
//重写了下面的代码
private int lockRunState() {
	int rs = runState;
  if((rs & 1) != 0){
    return awaitRunStateLock();
  }
  if(!U.compareAndSwapInt(this, RUNSTATE, rs, rs |= RSLOCK)){
    return awaitRunStateLock();
  }
  return rs;
}
```



**2.externalSubmit**

```java
```

