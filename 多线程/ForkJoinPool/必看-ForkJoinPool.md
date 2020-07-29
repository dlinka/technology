**核心概念**

```java
ForkJoinPool内部提供了一个为我们已经创建好的线程池-common
common中parallelism值为CPU核心数量-1,如果CPU核心数量是8,那么parallelism就等于7
parallelism等于7,WorkQueue数组长度等于16,7个工作队列,8个任务队列
WorkQueue数组偶数位表示任务队列,存放execute、submit方法的任务
WorkQueue数组奇数位表示工作队列,存放fork方法的任务,工作队列与ForkJoinWorkerThread绑定

//MASK表示掩码,位运算的时候使用
XXX_MASK
```

---

