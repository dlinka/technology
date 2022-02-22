```
Portainer
```

---

#### 常见问题

##### Repository和Registry

```
Repository类似Git仓库，里面存储一个项目所有的Tags
Registry就是Docker Hub
```

##### Docker运行nginx为什么要使用 daemon off

```
Docker默认会把pid=1的程序作为容器是否正在运行的依据，如果pid=1的进程结束，那么容器也会结束

如果使用nginx命令，那么nginx程序将是后台运行，这个时候nginx其实并不是pid为1的程序，而pid为1的程序其实是执行的bash，这个bash执行了nginx命令后就结束了，所以容器也就结束了
```



