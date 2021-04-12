1.put

```java
offer(e);
↓
↓
final ReentrantLock lock = this.lock;
//拿锁
lock.lock();
try {
    //向PriorityQueue队列中添加元素
    q.offer(e);
    //如果添加之后成为了第一个元素,就把leader线程置为null,同时唤醒一个等待池中的线程
    if (q.peek() == e) {
        leader = null;
        available.signal();
    }
    return true;
} finally {
    lock.unlock();
}
```

2.take

```java
final ReentrantLock lock = this.lock;
//拿锁
lock.lockInterruptibly();
try {
    //自旋
    for (;;) {
        //PriorityQueue队列第一个元素,但不出队
        E first = q.peek();
      	//PriorityQueue队列为空,则当前线程进入等待池
        if (first == null)
            available.await();
        else {
            long delay = first.getDelay(NANOSECONDS);
						//元素到期出队
            if (delay <= 0)
                return q.poll();
            first = null; //2
          	//有leader线程正在处理,则当前线程进入等待池,直到被唤醒
            if (leader != null)
                available.await();
            else {
                Thread thisThread = Thread.currentThread();
								//当前线程变成leader
              	leader = thisThread;
                try {
                    //leader线程进入等待池,等待到期时间重新竞争锁
                    available.awaitNanos(delay);
                } finally {
                    //如果leader线程没有被更改,就把leader置为null
                    if (leader == thisThread)
                        leader = null;
                }
            }
        }
    }
} finally {
    //如果leader等于null并且集合中还有元素,唤醒一个follower
    if (leader == null && q.peek() != null)
        available.signal();
    //释放锁
    lock.unlock();
}
```

2.1为什么这里要写first = null?

```java
假设n个线程同时调用take,这时会同时持有first元素
DelayQueue中有个10s后过期的元素e1,e1到期被一个线程拿走
那此时其他阻塞的线程还会持有这个已经被拿走了的e1,是没有意义的行为,防止内存泄漏
```