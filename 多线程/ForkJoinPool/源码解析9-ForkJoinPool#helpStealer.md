**helpStealer(WorkQueue w, ForkJoinTask<?> task)**

```java
WorkQueue[] ws = workQueues;
int oldSum = 0, checkSum, m;
if (ws != null && (m = ws.length - 1) >= 0 && w != null && task != null) {
    do {
        checkSum = 0;
        ForkJoinTask<?> subtask;
      	//j表示被窃取者
        //v表示窃取者
        WorkQueue j = w, v;
        descent: for (subtask = task; subtask.status >= 0; ) {
            //这个循环是寻找哪个WorkQueue窃取了任务
          	//随机一个奇数位置,每次循环+2
          	for (int h = j.hint | 1, k = 0, i; ; k += 2) {
              	...
                if ((v = ws[i = (h + k) & m]) != null) {
                  	//找到了窃取者
                    if (v.currentSteal == subtask) {
                        //记录这个窃取者的索引位,下次循环的时候可以直接定位到窃取者
                      	j.hint = i;
                        break;
                    }
                    checkSum += v.base;
                }
            }
            for (;;) {
                ForkJoinTask<?>[] a; int b;
                checkSum += (b = v.base);
              	//窃取者当前也可能正在join任务
                ForkJoinTask<?> next = v.currentJoin;
                ...
                //窃取者工作队列中此刻没有任务
              	//情况1:窃取者执行当前任务,并没有fork任务
              	//情况2:窃取者fork的所有任务又都被窃取了,而且窃取者正在join某一个fork出来的任务
              	if (b - v.top >= 0 || (a = v.array) == null) {
                  	//对应情况1:可能这个任务才开始执行,都还没有fork出来任务,所以重新循环
                    if ((subtask = next) == null)
                        break descent;
                  	//对应情况2:查找窃取者join的任务又被谁窃取了
                  	//说白了就是帮助窃取者的窃取者,递归操作
                    j = v;
                    break;
                }
              	//偷取窃取者base位置的任务去执行
                int i = (((a.length - 1) & b) << ASHIFT) + ABASE;
                ForkJoinTask<?> t = ((ForkJoinTask<?>) U.getObjectVolatile(a, i));
              	//窃取者base位置任务这时可能已被其他WorkQueue窃取,重新循环
              	if (v.base == b) {
                  	...
                    if (U.compareAndSwapObject(a, i, t, null)) {
                      	//base + 1
                      	v.base = b + 1;
                        //记录WorkQueue之前窃取的任务
                        ForkJoinTask<?> ps = w.currentSteal;
                        int top = w.top;
                        do {
                          	//记录WorkQueue当前窃取的任务
                            U.putOrderedObject(w, QCURRENTSTEAL, t);
                          	//执行任务
                            t.doExec();
                        //当前任务还没有结束 && 工作队列里面新增加了任务
                        //工作队列按照LIFO(栈)执行任务
                        } while (task.status >= 0 && w.top != top && (t = w.pop()) != null);
                        U.putOrderedObject(w, QCURRENTSTEAL, ps);
                        //当前工作队列中新增加了任务,则去执行自己的任务去,不要再帮助窃取者了
                        if (w.base != w.top)
                            return;
                    }
                }
            }
        }
    //对于上面情况1,假设窃取者执行的任务就是一直sleep(比较极端)
    //那么利用oldSum和checkSum这个判断也可以退出循环,不在浪费时间帮助了
    //留个问题:对于这种情况,一共执行几次循环(do/while)才会退出?
    } while (task.status >= 0 && oldSum != (oldSum = checkSum));
}
```

---
