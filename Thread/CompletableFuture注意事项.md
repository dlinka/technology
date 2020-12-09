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
        return "a";
    });
    ↓
    ↓
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> 0).thenApply((i) -> {
        try {
            TimeUnit.SECONDS.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "a";
    });
    //main线程会被阻塞
    System.out.println("main running");
    ↓
    ↓
    //强制使用多线程执行
    CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 0);
    CompletableFuture<String> future2 = future1.thenCompose(i -> CompletableFuture.supplyAsync(() -> "a" + 1));
    System.out.println(future2.get());
    
---
