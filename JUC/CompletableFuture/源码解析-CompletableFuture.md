#### CompletableFuture

##### 关键属性和关键方法

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
  	//结果
  	volatile Object result;
  	//利用CAS实现的无锁并发栈,stack表示栈顶元素
		volatile Completion stack;

		//使用自定义的线程池执行任务,推荐使用自定义线程池执行任务
  	public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor) {
    		return asyncSupplyStage(screenExecutor(executor), supplier);
		}
  
  	//自定义线程池执行AsyncSupply类型的任务
  	static <U> CompletableFuture<U> asyncSupplyStage(Executor e, Supplier<U> f) {
    		CompletableFuture<U> d = new CompletableFuture<U>();
    		e.execute(new AsyncSupply<U>(d, f));
    		return d;
		}
  
  	final void postComplete() {
        CompletableFuture<?> f = this;
      	//表示栈顶元素
      	Completion h;
        while ((h = f.stack) != null ||
               (f != this && (h = (f = this).stack) != null)) {
            CompletableFuture<?> d;
          	//栈顶元素的下一个元素
          	Completion t;
          	//出栈
            if (f.casStack(h, t = h.next)) {
                if (t != null) {
                    if (f != this) {
                        pushStack(h);
                        continue;
                    }
                    h.next = null;
                }
                f = (d = h.tryFire(NESTED)) == null ? this : d;
            }
        }
    }
  
  	final CompletableFuture<T> postFire(CompletableFuture<?> a, int mode) {
        if (a != null && a.stack != null) {
            if (mode < 0 || a.result == null)
                a.cleanStack();
            else
                a.postComplete();
        }
        if (result != null && stack != null) {
            if (mode < 0)
                return this;
            else
                postComplete();
        }
        return null;
    }
}
```

##### thenApply

```java
//如果任务没有结束,则使用前一个任务的线程执行
//如果任务已经结束,则当前哪个线程调用thenApply方法,就哪个线程执行
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn) {
  	return uniApplyStage(null, fn);
}

//使用线程池中的线程执行
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor) {
  	return uniApplyStage(screenExecutor(executor), fn);
}

private <V> CompletableFuture<V> uniApplyStage(Executor e, Function<? super T,? extends V> f) {
  	CompletableFuture<V> d =  new CompletableFuture<V>();
  	if (e != null || !d.uniApply(this, f, null)) {
    		UniApply<T,V> c = new UniApply<T,V>(e, d, this, f);
    		//入栈
    		push(c);
      	//尝试执行
    		c.tryFire(0);
  	}
  	return d;
}

final <S> boolean uniApply(CompletableFuture<S> a, Function<? super S,? extends T> f, UniApply<S,T> c) {
    Object r;
  	//如果前一个任务还没有完成,前一个任务的result就是null,这里直接返回false
    if (a == null || (r = a.result) == null || f == null)
        return false;
		if (result == null) {
        try {
          	//如果线程池存在,任务放入线程池执行,这里直接返回false
          	//如果线程池不存在,继续往下走
            if (c != null && !c.claim())
                return false;
            @SuppressWarnings("unchecked") S s = (S) r;
          	//使用当前线程执行任务
            completeValue(f.apply(s));
        } catch (Throwable ex) {
            completeThrowable(ex);
        }
    }
    return true;
}
```

##### 入栈

```java
final void push(UniCompletion<?,?> c) {
		if (c != null) {
      	//循环入栈,直到成功
    		while (result == null && !tryPushStack(c))
      			lazySetNext(c, null);
  	}
}

final boolean tryPushStack(Completion c) {
  	Completion h = stack;
  	lazySetNext(c, h);
  	return UNSAFE.compareAndSwapObject(this, STACK, h, c);
}

static void lazySetNext(Completion c, Completion next) {
  	UNSAFE.putOrderedObject(c, NEXT, next);
}
```
