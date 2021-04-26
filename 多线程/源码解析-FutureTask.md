state

```java
0=NEW //任务未执行
1=COMPLETING //任务完成了或者异常了,但是还没有保存任务结果或者异常
2=NORMAL //任务完成结果已保存
3=EXCEPTIONAL //任务异常结果已保存
4=CANCELLED //任务未执行或者任务执行中,调用cancel(false),取消任务但是不中断任务
5=INTERRUPTING //任务未执行或者任务执行中,调用cancel(true),取消任务并且准备中断任务
6=INTERRUPTED //中断任务
```

run

```java
任务未开始
  无论mayInterruptIfRunning值是什么,返回true
任务进行中
  mayInterruptIfRunning = true,返回true
  mayInterruptIfRunning = false,返回false
任务已完成
  无论mayInterruptIfRunning值是什么,返回false
```



