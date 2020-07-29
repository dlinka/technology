**awaitJoin(WorkQueue w, ForkJoinTask<?> task, long deadline)**

```java
int s = 0;
if (task != null && w != null) {
	//记录WorkQueue上一个join的任务
	ForkJoinTask<?> prevJoin = w.currentJoin;
	//WorkQueue的join的任务设置为当前任务
	U.putOrderedObject(w, QCURRENTJOIN, task);
	...
	for (;;) {
		//如果任务已经结束,直接退出循环
        if ((s = task.status) < 0)
            break;
      	...
      	//第1个条件是判断当前工作队列中任务是否全被窃取走了,如果全被窃取走了,去帮助窃取者执行任务
      	//第2个条件是在工作队列中查找这个任务,如果找到了就执行它,执行结束返回false
        else if (w.base == w.top || w.tryRemoveAndExec(task))
			//帮助窃取者去执行任务
          	helpStealer(w, task);
      	//再次检查任务是否结束
        if ((s = task.status) < 0)
            break;
        long ms, ns;
      	//默认是无限等待
        if (deadline == 0L)
            ms = 0L;
        ...
        //程序执行到这里,分2种情况
        //1种情况:这个WorkQueue弱爆了,还是阻塞等待吧,等待窃取者执行结束(tryRemoveAndExec)
        //2种情况:窃取者一直在执行这个任务,我也帮不上什么忙(比如任务一直sleep),还是阻塞等待吧
        //但是在等待之前,会进行一种补偿操作,如果满足补偿条件,线程才会阻塞,等待任务完成
      	if (tryCompensate(w)) {
            task.internalWait(ms);
          	//AC + 1
            U.getAndAddLong(this, CTL, AC_UNIT);
        }
    }
  	//还原回上一个join的任务
    U.putOrderedObject(w, QCURRENTJOIN, prevJoin);
}
return s;
```

