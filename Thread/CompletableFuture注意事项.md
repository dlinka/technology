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
    
    
