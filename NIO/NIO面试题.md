##### NIO使用Selector，为什么一定要配置configureBlocking(false)



##### IO操作为什么一定要有buffer

```
IO操作属于系统操作，系统操作就会涉及内核态和用户态的切换，这个开销是很大的
如果每次一有数据到来，就进行切换，无疑对系统的性能存在很大的影响，所以就需要引入buffer
```



##### NIO非阻塞场景下read什么时候返回-1，什么时候返回0

```
read返回-1，说明客户端数据发送完毕，并且主动关闭socket

read返回0有几种情况
第一种是当前SocketChannel中没有数据可以读
第二种是ByteBuffer中position等于limit了

```

