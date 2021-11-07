##### AsyncSupply

```java
static final class AsyncSupply<T> extends ForkJoinTask<Void> implements Runnable, AsynchronousCompletionTask {
		CompletableFuture<T> dep;
  	Supplier<T> fn;
  	
  	AsyncSupply(CompletableFuture<T> dep, Supplier<T> fn) {
    		this.dep = dep; this.fn = fn;
		}
		
		public void run() {
        CompletableFuture<T> d; Supplier<T> f;
        if ((d = dep) != null &&
            (f = fn) != null) {
            dep = null;
          	fn = null;
          	if (d.result == null) {
                //执行任务
								d.completeValue(f.get());
            }
            d.postComplete();
        }
		}
}
```

