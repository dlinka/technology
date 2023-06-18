#### FileChannelImpl类

```java
public class FileChannelImpl extends FileChannel {
  public long transferTo(long position, long count, WritableByteChannel target) {
    long n;
    //尝试直接传输,需要内核支持
    //Attempt a direct transfer, if the kernel supports it
    if ((n = transferToDirectly(position, icount, target)) >= 0)
      return n;

    //尝试通过mmap共享内存的方式进行可信通道的传输
    //Attempt a mapped transfer, but only to trusted channel types
    if ((n = transferToTrustedChannel(position, icount, target)) >= 0)
      return n;

    //否则通过传统方式进行传输,这种方式最慢
    //Slow path for untrusted targets
    return transferToArbitraryChannel(position, icount, target);
  }

  private long transferToDirectly(long position, int icount, WritableByteChannel target) {
    //获取当前文件通道和目标通道对象的fd
    int thisfdVal = IOUtil.fdVal(fd);
    int targetfdVal = IOUtil.fdVal(targetfd);
    //如果需要使用position锁,那么获取该锁调用transferToDirectlyInternal方法完成数据传送
    //通常transferToDirectlyNeedsPositionLock方法始终返回true
    if (nd.transferToDirectlyNeedsPositionLock()) {
      synchronized (positionLock) {
        long pos = position();
        try {
          return transferToDirectlyInternal(position, icount, target, targetFD);
        } finally {
          position(pos);
        }
      }
    }
  }

  private long transferToDirectlyInternal(long position, int icount, WritableByteChannel target, FileDescriptor targetFD) {
    do {
      //调用本地方法transferTo0完成传送
      n = transferTo0(fd, position, icount, targetFD);
    } while ((n == IOStatus.INTERRUPTED) && isOpen());
  }
  //如果操作系统不支持,那么将会返回-2
  //Transfers from src to dst, or returns -2 if kernel can't do that
  private native long transferTo0(FileDescriptor src, long position, long count, FileDescriptor dst);

  private long transferToTrustedChannel(long position, long count, WritableByteChannel target) {
    long remaining = count;
    while (remaining > 0L) {
      //设置了最大mmap的大小为MapPED_TRANSFER_SIZE 8M
      //如果需要传送的文件数据大于这个值,那么需要分阶段映射
      long size = Math.min(remaining, MAPPED_TRANSFER_SIZE);
      try {
        //获取当前文件映射缓冲区
        MappedByteBuffer dbb = map(MapMode.READ_ONLY, position, size);
        try {
					//调用目标通道进行数据写入
          int n = target.write(dbb);
        } finally {
          //写入完成,结束内存映射
          unmap(dbb);
        }
      }
    }
    return count - remaining;
  }
  public MappedByteBuffer map(MapMode mode, long position, long size) throws IOException {
    try {
      //调用本地方法完成映射
      addr = map0(imode, mapPosition, mapSize);
    }
    
    //映射结束处理对象
    Unmapper um = new Unmapper(addr, mapSize, isize, mfd);
    //根据模式构建只读MapPedByteBufferR或者读写MapPedByteBuffer
    if ((!writable) || (imode == MAP_RO)) {
      return Util.newMappedByteBufferR(isize,
                                       addr + pagePosition,
                                       mfd,
                                       um);
    } else {
      return Util.newMappedByteBuffer(isize,
                                      addr + pagePosition,
                                      mfd,
                                      um);
    }
  }
  private native long map0(int prot, long position, long length) throws IOException;
  
  
  private long transferToArbitraryChannel(long position, int icount, WritableByteChannel target) throws IOException {
    while (tw < icount) {
      //将文件数据读入到堆内缓冲区中
    	int nr = read(bb, pos);
      //从读模式切换到写模式
      bb.flip();
      //将文件数据写入到堆内缓冲区中
      int nw = target.write(bb);
      //清空缓冲区,方便下一次读取
      bb.clear();
    }
  }
} 
```

### Linux

#### transferTo0

```c
JNIEXPORT jlong JNICALL Java_sun_nio_ch_FileChannelImpl_transferTo0(JNIEnv *env,
                                                                    jobject this,
                                                                    jint srcfd,
                                                                    jlong position,
                                                                    jlong count,
                                                                    jint dstfd)
{
  off64_t offset = (off64_t)position;
  jlong n = sendfile64(dstfd,          // 目标描述符
                       srcfd,          // 源描述符
                       &offset,        //源传送偏移量
                       (size_t)count); // 传送大小
  ...
    return n;
}
```

#### map0

```c
JNIEXPORT jlong JNICALL Java_sun_nio_ch_FileChannelImpl_map0(JNIEnv *env,
                                                             jobject this,
                                                             jint prot,
                                                             jlong off,
                                                             jlong len){
  void *mapAddress = 0;
  jobject fdo = (*env)->GetObjectField(env, this, chan_fd);
  jint fd = fdval(env, fdo);
  int protections = 0;
  int flags = 0;
  //设置映射标志位
  if (prot == sun_nio_ch_FileChannelImpl_MAP_RO) {
    //只读映射
    protections = PROT_READ;
    flags = MAP_SHARED;
  } else if (prot == sun_nio_ch_FileChannelImpl_MAP_RW) {
    //读写映射
    protections = PROT_WRITE | PROT_READ;
    flags = MAP_SHARED;
  } else if (prot == sun_nio_ch_FileChannelImpl_MAP_PV) {
    //私有映射
    protections =  PROT_WRITE | PROT_READ;
    flags = MAP_PRIVATE;
  }
  //使用mmap64函数进行映射
  mapAddress = mmap64(
    0,                    //传入期望映射地址为0,表明让内核决定映射起始虚拟地址
    len,                  //映射的长度
    protections,          //映射地址权限:读、读写
    flags,                //是否为私有映射
    fd,                   //映射的文件描述符
    off);                 //映射的文件数据偏移量

  return ((jlong) (unsigned long) mapAddress);
}
```

