# CompletableFuture

```
JDK之前有个Future接口，但是调用其get方法时，线程会被阻塞，违反了异步编程的初衷
而且如果需要判断任务是否结束，还需要进行轮询处理，执行一个大任务，就会导致CPU大部分时间都在空转，很影响性能
```

---

#### CompletableFuture

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {

  //任务的执行结果
  volatile Object result;
  volatile Completion stack;

  //推荐使用自定义线程池执行任务
  public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                     Executor executor) {
    return asyncSupplyStage(screenExecutor(executor), supplier);
  }

  static <U> CompletableFuture<U> asyncSupplyStage(Executor e,
                                                   Supplier<U> f) {
    CompletableFuture<U> d = new CompletableFuture<U>();
    //放到线程池中执行
    e.execute(new AsyncSupply<U>(d, f));
    return d;
  }

  //设置返回值
  final boolean completeValue(T t) {
    return UNSAFE.compareAndSwapObject(this, RESULT, null, (t == null) ? NIL : t);
  }

  public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn,
                                                 Executor executor) {
    return uniApplyStage(screenExecutor(executor), fn);
  }

  private <V> CompletableFuture<V> uniApplyStage(Executor e,
                                                 Function<? super T,? extends V> f) {
    CompletableFuture<V> d =  new CompletableFuture<V>();
    if (e != null || !d.uniApply(this, f, null)) {
      UniApply<T,V> c = new UniApply<T,V>(e, d, this, f);
      push(c);
      c.tryFire(SYNC);
    }
    return d;
  }

}
```

#### AsyncSupply

```java
static final class AsyncSupply<T> extends ForkJoinTask<Void>
  implements Runnable, AsynchronousCompletionTask {
  CompletableFuture<T> dep; Supplier<T> fn;
  AsyncSupply(CompletableFuture<T> dep, Supplier<T> fn) {
    this.dep = dep; this.fn = fn;
  }

  public void run() {
    CompletableFuture<T> d; Supplier<T> f;
    if ((d = dep) != null && (f = fn) != null) {
      dep = null; fn = null;
      if (d.result == null) {
        //执行任务
        d.completeValue(f.get());
      }
      //执行完当前任务，执行后续任务
      d.postComplete();
    }
  }
}
```

