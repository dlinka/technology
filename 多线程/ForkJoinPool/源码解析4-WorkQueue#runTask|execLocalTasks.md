**1.runTask(ForkJoinTask<?> task)**

```java
if (task != null) {
	//将scanState最低位变成0,表示WorkQueue正在执行任务
	scanState &= ~SCANNING;
  //任务执行
  (currentSteal = task).doExec();
	U.putOrderedObject(this, QCURRENTSTEAL, null);
  execLocalTasks(); //2
  ...
  //任务执行完毕,恢复scanState
	scanState |= SCANNING;
  ...
}
↓
↓
//进入doExec方法,也就是进入ForkJoinTask#doExec()方法
completed = exec();
↓
↓
//进入exec方法,也就是进入用户自定义的RecursiveAction#exec()方法
//真正的任务就开始执行了...
compute();
```

**U.putOrderedObject(this, QCURRENTSTEAL, null);**

```java
//为什么不直接使用currentSteal = null?
currentSteal首先使用volatile修饰,直接赋值会在写操作前后插入StoreStore屏障
putOrderedObject可以保证指令不重排序,但是不保证可见性,所以这样写入的性能更高
```

**2.execLocalTasks()**

```java
int b = base, m, s;
ForkJoinTask<?>[] a = array;
//本地工作队列中存在任务
if (b - (s = top - 1) <= 0 && a != null && (m = a.length - 1) >= 0) {
  //默认是LIFO_QUQUE(工作队列就是一个栈)
  if ((config & FIFO_QUEUE) == 0) {
    for (ForkJoinTask<?> t;;) {
      //top - 1位置的任务出队,也就是栈顶的任务出队
      if ((t = (ForkJoinTask<?>) U.getAndSetObject(a, ((m & s) << ASHIFT) + ABASE, null)) == null)
        break;
      //更新top的值
      U.putOrderedInt(this, QTOP, s);
      //执行任务
      t.doExec();
      //工作队列中不存在任务了,退出循环
      if (base - (s = top - 1) > 0)
        break;
    }
  }
  else
    //FIFO
    pollAndExecAll();
}
```

---
