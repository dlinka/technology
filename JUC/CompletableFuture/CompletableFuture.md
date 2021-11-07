```
JDK之前有个Future接口,但是调用其get方法时,线程会被阻塞,这违反了异步编程的初衷
而且如果需要判断任务是否结束,需要轮询处理,如果执行一个大任务,CPU大部分时间都在空转,很影响性能
CompletableFuture是JDK1.8加入的,其利用回调函数完美解决了上面说的两个问题,它可以在任务结束后会自动执行某些功能

使用CompletableFuture如果执行的任务很耗时,那么请使用自定义的Executor,或者在方法末尾加上future.get(),等待执行结束.因为CompletableFuture默认使用ForkJoinPool,而ForkJoinPool里面的线程都是daemon线程,main线程跑完了,虚拟机也就结束了.

thenApply和thenApplyAsync方法的区别
先说thenApplyAsync,一定会使用线程池中的线程执行,可能是同一个,也可能线程池创建一个新的线程执行.
再说thenApply,如果任务没有结束,使用前一个线程执行,如果任务已经结束,当前哪个线程调用thenApply方法,就哪个线程执行.
```

---

| 方法                                                         |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| runAsync(Runnable runnable)                                  | 使用ForkJoinPool执行一个没有返回值的任务                     |
| runAsync(Runnable runnable, Executor executor)               | 使用自定义线程池执行一个没有返回值的任务(推荐)               |
| supplyAsync(Supplier<U> supplier)                            | 使用ForkJoinPool执行一个有返回值的任务                       |
| supplyAsync(Supplier<U> supplier, Executor executor)         | 使用自定义线程池执行一个有返回值的任务(推荐)                 |
| thenRun(Runnable action)                                     | 任务1执行完后执行任务2,任务2不需要任务1的结果,任务2也没有返回值 |
| thenAccept(Consumer<? super T> action)                       | 任务1执行完后执行任务2,任务2需要任务1的结果,任务2也没有返回值 |
| thenApply(Function<? super T,? extends U> fn))               | 任务1执行完后执行任务2,任务2需要任务1的结果,任务2有返回值    |
| runAfterBoth(CompletionStage<?> other, Runnable action)      | 任务1和任务2执行完后,执行任务3,任务3不需要任务1和任务2结果,任务3也没有返回值 |
| thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action) | 任务1和任务2执行完后,执行任务3,任务3需要任务1和任务2结果,任务3没有返回值 |
| thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn) | 任务1和任务2执行完后,执行任务3,任务3需要任务1和任务2结果,任务3有返回值 |
| allOf((CompletableFuture<?>... cfs))                         | 等待所有任务执行结束                                         |
| anyOf((CompletableFuture<?>... cfs))                         | 只要任意一个任务执行完成就可以                               |
| runAfterEither(CompletionStage<?> other, Runnable action)    | 任务1和任务2中的其中一个执行完后,执行任务3,任务3不需要任务1或者任务2的结果,任务3也没有返回值 |
| acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action) | 任务1和任务2中的其中一个执行完后,执行任务3,任务3需要任务1或者任务2的结果,任务3没有返回值 |
| applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn) | 任务1和任务2中的其中一个执行完后,执行任务3,任务3需要任务1或者任务2的结果,任务3有返回值 |
| exceptionally(Function<Throwable, ? extends T> fn))          | 如果任务1抛出异常,在任务2中进行处理,任务2有返回值            |
| handle(BiFunction<? super T, Throwable, ? extends U> fn)     | 任务1执行完成后,执行任务2,如果任务1执行正常,任务2拿到任务1的返回值,如果任务1抛出异常,任务2拿到任务1抛出的异常 |
| thenCompose(Function<? super T, ? extends CompletionStage<U>> fn) | 任务1执行完成后,执行任务2,类似"thenApply+flatMap"            |

---

**线程执行的不确定性**

```java
CompletableFuture<Integer> cf = CompletableFuture.supplyAsync(() -> {
    //ForkJoinPool.commonPool-worker-1
    Thread.currentThread().getName();
		TimeUnit.SECONDS.sleep(0);
    return 1;
}).thenApply(i -> {
    //ForkJoinPool.commonPool-worker-1或者main
    Thread.currentThread().getName();
    return i;
});
```

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 1).thenApply(i -> {
    TimeUnit.SECONDS.sleep(1000);    
    return i;
});
//main线程被阻塞
```

---

**依赖前一个任务结果再去进行计算可以使用thenApply,为什么还会有thenCompose方法呢?**

```java
//compose2依赖compose1
CompletableFuture<Integer> compose1() {
    return CompletableFuture.supplyAsync(() -> 0);
}
CompletableFuture<String> compose2(Integer i) {
    return CompletableFuture.supplyAsync(() -> "a " + i);
}

//使用thenApply方法返回值没有使用thenCompose方法返回值友好
CompletableFuture<CompletableFuture<String>> future = compose1().thenApply(i -> compose2(i));
CompletableFuture<String> future = compose1().thenCompose(i -> compose2(i));
```

---

