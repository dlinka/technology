epoll_create创建eventpoll

```c
long sys_epoll_create(int size) {
  struct eventpoll *ep;
  ep_alloc(&ep); //为ep分配内存并进行初始化
  fd = anon_inode_getfd("[eventpoll]", &eventpoll_fops, ep, O_RDWR | (flags & O_CLOEXEC));
  return fd;
}
```

epoll_ctl添加epitem

```c
long sys_epoll_ctl(int epfd, int op, int fd, struct epoll_event __user *event) {
	struct file *file,*tfile;
  struct eventpoll *ep;
  struct epoll_event epds;

	//判断参数的合法性，将 __user *event复制给epds。
	if(ep_op_has_event(op) && copy_from_user(&epds, event, sizeof(struct epoll_event)))
    goto error_return;

	file  = fget (epfd); //eventpoll的fd对应的文件对象
	tfile = fget(fd);    //fd对应的文件对象

	ep = file->private->data; //获取eventpoll

	//防止重复添加，在eventpoll的红黑树中查找是否已经存在这个fd
  epi = epi_find(ep, tfile, fd);

  switch(op)
  {
    //增加监听一个fd
    case EPOLL_CTL_ADD:
      if (!epi)
      {
        epds.events |= EPOLLERR | POLLHUP;     //默认包含POLLERR和POLLHUP事件
        error = ep_insert(ep, &epds, tfile, fd);  //在ep的红黑树中插入这个fd对应的epitem结构体
      }
  }
	return error;
}
↓
↓
int ep_insert(struct eventpoll *ep, struct epoll_event *event, struct file *tfile, int fd) {
  //初始化epitem
  INIT_LIST_HEAD(&epi->rdllink);
  ...
  
  //将epitem插入红黑树
	ep_rbtree_insert(ep, epi);
  
  /**
	 * 如果要监视的fd状态改变，并且还没有加入到就绪链表中，则将当前的epitem加入到就绪链表中
	 * 如果有进程正在等待该文件的状态就绪，则唤醒等待的进程
	 */
  if ((revents & event->events) && !ep_is_linked(&epi->rdllink)) {
    list_add_tail(&epi->rdllink, &ep->rdllist);

    if (waitqueue_active(&ep->wq))
      wake_up_locked(&ep->wq);

    if (waitqueue_active(&ep->poll_wait))
      pwake++;
  }
}
```

epoll_wait

```c
```





##### 数据结构

```c
struct eventpoll {
  /**
   * 等待队列可以看作保存进程的容器
   * 当阻塞进程时，将进程放入等待队列，当唤醒进程时，从等待队列中取出进程
   */
  wait_queue_head_t  wq;
  
	/**
	 * 红黑树的根节点
	 * 这个树可以看作存储着所有添加到epoll中的需要监控的Socket
	 */
  struct rb_root     rbr;
  
  /**
   * 双链表
   * 这个链表可以看作存放着将要通过epoll_wait返回给用户的就绪Socket
   */
  struct list_head   rdlist;
}

struct epitem {
  struct rb_node       rbn;     //红黑树节点
  struct list_head     rdllink; //双向链表节点
  struct epoll_filefd  ffd;     //事件句柄信息
  struct eventpoll     *ep;     //指向其所属的eventpoll对象
  struct epoll_event   event;   //期待发生的事件类型
}
```

