#### mmap和sendFile

##### 传统IO

![传统IO](/Users/dlinka/GitHub/technology/操作系统/传统IO.png)

```
经历4次上下文切换
用户态 -> 内核态 -> 用户态 -> 内核态 -> 用户态

经历4次拷贝
磁盘文件DMA拷贝到内核缓冲区
内核缓冲区CPU拷贝到用户缓冲区
用户缓冲区CPU拷贝到socket缓冲区
socket缓冲区DMA拷贝到协议引擎
```

##### mmap

![mmap](/Users/dlinka/GitHub/technology/操作系统/mmap.png)

```
经历4次上下文切换
用户态 -> 内核态 -> 用户态 -> 内核态 -> 用户态

但是只有3次拷贝
磁盘文件DMA拷贝到内核缓冲区
内核缓冲区CPU拷贝到socket缓冲区
socket缓冲区DMA拷贝到协议引擎

相对上下文切换次数没有变更，但是少了一次拷贝，可以直接从内核缓冲区拷贝到socket缓冲区
```

##### sendFile

![sendFile](/Users/dlinka/GitHub/technology/操作系统/sendFile.png)

```
只有2次上下文切换
用户态 -> 内核态 -> 用户态

只有2次拷贝
磁盘文件DMA拷贝到内核缓冲区
内核缓冲区DMA拷贝到协议引擎
```

