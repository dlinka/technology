线程状态

NEW

    new Thread()就是这个状态

RUNNABLE
    
    官方API解释这个状态代表线程正在Java虚拟机中运行
    但是线程可能正在等待CPU,这句话什么意思呢?我的理解是OS的可运行状态在Java里面也算是RUNNABLE状态
        
BLOCKED

    线程进入锁池就进入这个状态

WAITING

    线程进入等待池就进入这个状态

TERMINATED

    线程执行结束

---
