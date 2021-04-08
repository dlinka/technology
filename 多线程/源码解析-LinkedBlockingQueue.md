1.put

```java
int c = -1;
Node<E> node = new Node<E>(e);
final ReentrantLock putLock = this.putLock;
final AtomicInteger count = this.count;
//拿到putLock锁
putLock.lockInterruptibly();
try {
    //容器元素到达容量限制阻塞
  	while (count.get() == capacity) {
  			notFull.await();
  	}
  	enqueue(node);
  	//增加集合中元素数量
  	c = count.getAndIncrement();
  	//如果容器未到容量限制可以继续唤醒生产者生产
  	//这一步很关键因为消费者只会唤醒一个生产者
  	if (c + 1 < capacity)
    		notFull.signal();
} finally {
    putLock.unlock();
}
//唤醒消费者
if (c == 0)
    signalNotEmpty();
↓
↓
//enqueue
//最后一个节点指向node,最后一个节点变成node
last = last.next = node;
```

2.take

```java
E x;
int c = -1;
final AtomicInteger count = this.count;
final ReentrantLock takeLock = this.takeLock;
//拿到takeLock锁
takeLock.lockInterruptibly();
try {
  	//容器没有元素线程阻塞
    while (count.get() == 0) {
        notEmpty.await();
    }
    x = dequeue();
    c = count.getAndDecrement();
  	//如果元素个数大于1就可以唤醒其他消费者
    //这一步很关键因为生产者只会唤醒一个消费者
    if (c > 1)
        notEmpty.signal();
} finally {
    takeLock.unlock();
}
//唤醒一个生产者
if (c == capacity)
    signalNotFull();
return x;
↓
↓
Node<E> h = head;
Node<E> first = h.next;
//注释上说这行有助于help GC,有知道怎么有助于的可以讲讲
h.next = h;
head = first;
E x = first.item;
first.item = null;
return x;
```

---

```java
通过上面的源码知道LinkedBlockingQueue使用了两把锁,一把是对生产者,一把是对消费者
最开始我也疑问这么设计不会产生线程不安全吗
其实并不会因为在put的时候使用的是last变量,而在take的时候使用的是head变量,很巧妙的设计
```



