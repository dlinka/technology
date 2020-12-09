使用exceptionally方法的注意事项

    CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
        int throwException = 12 / 0;
        return 10;
    });
    //这种方式是获取不到20返回值的
    //因为这段代码执行的时候前面的异常已经抛出了
    future.exceptionally((ex) -> {
        return 20;
    });
    ↓
    ↓
    CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
        int throwException = 12 / 0;
        return 10;
    }).exceptionally((ex) -> {
        return 20;
    });
    
线程调用不确定可能导致线程阻塞

    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
        //ForkJoinPool.commonPool-worker-1
        System.out.println(Thread.currentThread().getName());
        //开启这段代码,thenApply中打印:ForkJoinPool.commonPool-worker-1
        //关闭这段代码,thenApply中打印:main
        try {
            TimeUnit.SECONDS.sleep(0);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return 0;
    }).thenApply((i) -> {
        System.out.println(Thread.currentThread().getName());
        return "a" + i;
    });
    ↓
    ↓
    //main线程被阻塞的情况
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> 0).thenApply((i) -> {
        try {
            TimeUnit.SECONDS.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "a" + i;
    });
    System.out.println("main running");

theCompose的使用场景

    //依赖前一个任务结果再去进行计算可以使用thenApply,为什么还会有thenCompose?
    
    static CompletableFuture<Integer> compose1() {
        return CompletableFuture.supplyAsync(() -> 0);
    }
    static CompletableFuture<String> compose2(Integer i) {
        return CompletableFuture.supplyAsync(() -> "a " + i);
    }
    //使用thenApply返回值没有使用thenCompose友好
    CompletableFuture<CompletableFuture<String>> future = compose1().thenApply(i -> compose2(i));
    CompletableFuture<String> future = compose1().thenCompose(i -> compose2(i));

---
