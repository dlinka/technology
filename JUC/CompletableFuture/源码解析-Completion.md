##### Completion

```java
abstract static class Completion extends ForkJoinTask<Void> implements Runnable, AsynchronousCompletionTask {
		//无锁并发栈中下一个元素的引用
  	volatile Completion next;

  	abstract CompletableFuture<?> tryFire(int mode);

	  public final void run() { tryFire(1); }
}
```

##### UniCompletion

```java
abstract static class UniCompletion<T,V> extends Completion {
		//执行任务的线程池
  	Executor executor;
  	//dep的执行依赖于src
		CompletableFuture<V> dep;
  	CompletableFuture<T> src;

  	UniCompletion(Executor executor, CompletableFuture<V> dep, CompletableFuture<T> src) {
				this.executor = executor;
      	this.dep = dep;
      	this.src = src;
		}
  
  	final boolean claim() {
        Executor e = executor;
        if (compareAndSetForkJoinTaskTag((short)0, (short)1)) {
            if (e == null)
                return true;
            executor = null;
            e.execute(this);
        }
        return false;
    }
}
```

##### UniApply

```java
static final class UniApply<T,V> extends UniCompletion<T,V> {
  	Function<? super T,? extends V> fn;
  	
  	UniApply(Executor executor, CompletableFuture<V> dep, CompletableFuture<T> src, Function<? super T,? extends V> fn) {
				super(executor, dep, src);
      	this.fn = fn;
    }
  
  	final CompletableFuture<V> tryFire(int mode) {
        CompletableFuture<V> d; CompletableFuture<T> a;
      	if ((d = dep) == null || !d.uniApply(a = src, fn, mode > 0 ? null : this))
            return null;
        dep = null; src = null; fn = null;
        return d.postFire(a, mode);
    }
}
```