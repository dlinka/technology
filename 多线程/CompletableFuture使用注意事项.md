### exceptionally调用注意事项

```java
CompletableFuture<Integer> cf = CompletableFuture.supplyAsync(() -> {
    int i = 12 / 0;
    return 1;
});
//这段代码执行的时候前面的异常已经抛出了
cf.exceptionally((ex) -> {
    return 2;
});
↓
↓
CompletableFuture<Integer> cf = CompletableFuture.supplyAsync(() -> {
    int i = 12 / 0;
    return 1;
}).exceptionally(ex -> {
    return 2;
});
```

### 线程执行的不确定性

```java
CompletableFuture<Integer> cf = CompletableFuture.supplyAsync(() -> {
    //ForkJoinPool.commonPool-worker-1
    System.out.println(Thread.currentThread().getName());
    try {
        TimeUnit.SECONDS.sleep(0);
    } catch (InterruptedException e) {
    }
    return 1;
}).thenApply(i -> {
    //打印ForkJoinPool.commonPool-worker-1或者main
    System.out.println(Thread.currentThread().getName());
    return 2;
});
```

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 1).thenApply(i -> {
    try {
        TimeUnit.SECONDS.sleep(1000);
    } catch (InterruptedException e) {
    }
    return 2;
});
//main线程被阻塞
System.out.println("main");
```



### thenCompose方法

```java
//依赖前一个任务结果再去进行计算可以使用thenApply,为什么还会有thenCompose?

static CompletableFuture<Integer> compose1() {
    return CompletableFuture.supplyAsync(() -> 0);
}
static CompletableFuture<String> compose2(Integer i) {
    return CompletableFuture.supplyAsync(() -> "a " + i);
}
//使用thenApply方法返回值没有使用thenCompose方法返回值友好
CompletableFuture<CompletableFuture<String>> future = compose1().thenApply(i -> compose2(i));
CompletableFuture<String> future = compose1().thenCompose(i -> compose2(i));
```

---

参考:

[Javadoop](https://www.javadoop.com/post/completable-future)

---