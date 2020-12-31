##### 执行顺序按照调用顺序

```java
//whenComplete执行会在exceptionally前执行
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    int i = 2 / 0;
    return 1;
}).whenComplete((i, ex) -> {
    System.out.println("whenComplete entry");
    System.out.println("i : " + i);
    System.out.println("ex : " + ex);
    System.out.println("whenComplete exit");
}).exceptionally((ex) -> {
    System.out.println("exceptionally entry");
    System.out.println("ex : " + ex);
    System.out.println("exceptionally exit");
    return 20;
});
```



##### 返回值不可变

```java
CompletableFuture<LocalDate> future = CompletableFuture.supplyAsync(() -> {
    return LocalDate.of(1988, 11, 8);
}).whenComplete((v, ex) -> {
    v = LocalDate.MAX;
});
//1988-11-08
System.out.println(future.join());
↓
↓
//如果需要返回值改变,可以使用handle方法
CompletableFuture<LocalDate> future = CompletableFuture.supplyAsync(()->{
		return LocalDate.of(1988, 11, 8);
}).handle((v, ex)->{
    v = LocalDate.MAX;
    return v;
});
```



##### join方法

```java
join和get都是阻塞获取值
join不需要手动抛出异常
get需要手动抛出异常
  ExecutionException, InterruptedException
```



##### complete方法

```java
CompletableFuture<Integer> future = new CompletableFuture<>();
new Thread(()->{
    try {
        TimeUnit.SECONDS.sleep(5);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    future.complete(27);
}).start();
System.out.println(future.join();
```



##### completedFuture静态方法

```java
//可以直接生成一个已完成的CompletableFuture
CompletableFuture<Integer> future = CompletableFuture.completedFuture(27);
System.out.println(future.join());
```



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



##### 线程执行的不确定性

thenApply使用同一个线程执行

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    //ForkJoinPool.commonPool-worker-1
    System.out.println(Thread.currentThread().getName());
    try {
        TimeUnit.SECONDS.sleep(0);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return 0;
}).thenApply((i) -> {
    //ForkJoinPool.commonPool-worker-1
    System.out.println(Thread.currentThread().getName());
    return "a" + i;
});
```

thenApply使用main线程执行

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    //ForkJoinPool.commonPool-worker-1
    System.out.println(Thread.currentThread().getName());
    return 0;
}).thenApply((i) -> {
    //main
    System.out.println(Thread.currentThread().getName());
    return "a" + i;
});
```

main线程被阻塞

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> 0).thenApply((i) -> {
    try {
        TimeUnit.SECONDS.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "a";
});
//这里不会立即被打印
System.out.println("main running");
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