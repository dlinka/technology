线程状态

NEW

    new Thread()就是这个状态

RUNNABLE
    
    官方API解释这个状态代表线程正在Java虚拟机中运行,但是线程可能正在等待CPU
    后面这句话什么意思呢?我的理解是OS中的可运行状态在Java里面也算是RUNNABLE状态了

    通过上面的定义引申出来,线程执行I/O,他的状态应该也是RUNNABLE,因为线程一直在JVM中运行
    这点可以知道Java的线程状态和OS的线程状态是完全的两码事

BLOCKED

    线程进入锁池就进入这个状态

WAITING

    线程进入等待池就进入这个状态

TERMINATED

    线程执行结束

---
