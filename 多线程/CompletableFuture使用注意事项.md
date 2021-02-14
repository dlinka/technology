##### exceptionally调用注意事项

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    int i = 12 / 0;
    return 1;
});
//下面这段代码执行的时候前面的异常已经抛出了
future.exceptionally((ex) -> {
    return 2;
});
↓
↓
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    int throwException = 12 / 0;
    return 1;
}).exceptionally((ex) -> {
    return 2;
});
```



### 线程执行的不确定性

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    //ForkJoinPool.commonPool-worker-1
    System.out.println(Thread.currentThread().getName());
    try {
        TimeUnit.SECONDS.sleep(0);
    } catch (InterruptedException e) {
    }
    return 0;
}).thenApply((i) -> {
    //下面打印ForkJoinPool.commonPool-worker-1或者main
    System.out.println(Thread.currentThread().getName());
    return "a";
});
```

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> 0).thenApply((i) -> {
    try {
        TimeUnit.SECONDS.sleep(10000);
    } catch (InterruptedException e) {
    }
    return "a";
});
//main线程被阻塞
//这里不会立即被打印
System.out.println("main");
```

##### theCompose方法

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