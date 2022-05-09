##### 1.offer

```java
public boolean offer(E e) {
  final ReentrantLock lock = this.lock;
  lock.lock();
  try {
    q.offer(e); //往优先级队列中添加元素
    if (q.peek() == e) { //如果添加之后成为了第一个元素，唤醒一个等待池中的线程重新竞争leader
      leader = null;
      available.signal();
    }
    return true;
  } finally {
    lock.unlock();
  }
}
```

##### 2.take

```java
public E take() throws InterruptedException {
  final ReentrantLock lock = this.lock;
  lock.lockInterruptibly();
  try {
    for (;;) { //死循环
      E first = q.peek(); //拿到优先级队列第一个元素
      if (first == null) //如果优先级队列为空，则当前线程进入等待池
        available.await();
      else {
        long delay = first.getDelay(NANOSECONDS);
        if (delay <= 0)  //如果delay小于等于0表示元素到期可以出队，元素出队
          return q.poll();
        first = null; //2.1
        if (leader != null) //如果leader线程不为空，则当前线程进入等待池，直到被唤醒
          available.await();
        else {
          Thread thisThread = Thread.currentThread();
          leader = thisThread; //将当前线程变成leader线程
          try {
            available.awaitNanos(delay); //leader线程进入等待池，等待到期时间唤醒重新竞争锁
          } finally {
            if (leader == thisThread) //如果leader线程没有被更改，就把leader置为null
              leader = null;
          }
        }
      }
    }
  } finally {
    if (leader == null && q.peek() != null) //如果leader等于null并且集合中还有元素，唤醒一个follower
      available.signal();
    lock.unlock();
  }
}
```

##### 2.1.为什么这里要写first = null?

```java
假设n个线程同时调用take，这时n个线程会同时持有first元素指向的引用
DelayQueue中持有一个10s后过期的元素E1，某一个线程拿走E1后，此时其他等待的线程还会持有这个已经被拿走了的E1
为了防止内存泄漏，这里强制first = null
```