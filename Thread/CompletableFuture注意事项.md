exceptionally

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
    
thenApply

    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName());
        //开启这段代码,下面thenApply打印:ForkJoinPool.commonPool-worker-1
        //关闭这段代码,下面thenApply打印:main
        try {
            TimeUnit.SECONDS.sleep(0);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return 0;
    }).thenApply((i) -> {
        System.out.println(Thread.currentThread().getName());
        return "a";
    });
    
---
