1.current

```java
//多线程每个线程都会调用localInit方法
if (UNSAFE.getInt(Thread.currentThread(), PROBE) == 0)
	localInit();
//返回单例ThreadLocalRandom对象
return instance;
```

2.localInit

```java
int p = probeGenerator.addAndGet(PROBE_INCREMENT);
int probe = (p == 0) ? 1 : p;
long seed = mix64(seeder.getAndAdd(SEEDER_INCREMENT));
Thread t = Thread.currentThread();
UNSAFE.putLong(t, SEED, seed);
UNSAFE.putInt(t, PROBE, probe);
```

---

