CPU核总数 = CPU个数*CPU核数  
逻辑CPU总数 = CPU核总数*超线程数

mac

    sysctl machdep.cpu
    ...
    //CPU核总数
    machdep.cpu.core_count: 2
    //逻辑CPU总数
    machdep.cpu.thread_count: 4
    
linux

    cat /proc/cpuinfo|grep "physical id"|sort|uniq
    //2个物理CPU
    physical id     : 0
    physical id     : 1
    
    cat /proc/cpuinfo|grep "cpu cores"|sort|uniq
    //每个物理CPU上有4个核
    cpu cores       : 4
    
    cat /proc/cpuinfo|grep "core id"|sort|uniq
    //CPU核总数为8
    core id         : 0
    core id         : 1
    core id         : 2
    core id         : 3
    core id         : 4
    core id         : 5
    core id         : 6
    core id         : 7
    
    cat /proc/cpuinfo|grep "siblings"|sort|uniq
    //每个物理CPU上有8个逻辑CPU
    siblings        : 8
    
    cat /proc/cpuinfo|grep "processor"|sort|uniq
    //逻辑CPU总数为16
    processor       : 0
    processor       : 1
    processor       : 10
    processor       : 11
    processor       : 12
    processor       : 13
    processor       : 14
    processor       : 15
    processor       : 2
    processor       : 3
    processor       : 4
    processor       : 5
    processor       : 6
    processor       : 7
    processor       : 8
    processor       : 9

    processor //逻辑CPU的id
    physical id //物理CPU的id
    core id //CPU核id
    cpu cores //这个物理CPU上的CPU核数
    siblings //这个物理CPU上的逻辑CPU数
    
---
