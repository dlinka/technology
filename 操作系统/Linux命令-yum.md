```
一般镜像源都提供了centosplus,cloud,extras,fasttrack,os,updates等软件库,最好认的软件库就是os(系统默认的软件)与updates(软件升级版本)
yum源最重要的就是repodata目录,该目录就是分析RPM软件后所产生的软件属性相依数据放置处!当你要找repository所在网址时,最重要的就是该网址底下一定要有个名为repodata的目录存在!
```

---

#### 变量

1.$releasever:当前系统的发行版本

```shell
rpm -qi centos-release
↓
↓
Name        : centos-release
#7就是$releasever的值,也就是当前系统的发行版本
Version     : 7
...
```

2.$basearch:CPU架构

```shell
arch
↓
↓
#x86_64就是$basearch的值
x86_64
```

#### 全局设置

路径:/etc/yum.conf

```shell
[main]
#缓存目录
cachedir=/var/cache/yum/$basearch/$releasever
logfile=/var/log/yum.log
#如果你设置了多个repository,同一软件在不同的repository中同时存在,yum会安装最新的那个版本,默认就是newest
pkgpolicy=newest
```

#### 源配置

路径:/etc/yum.repos.d/*.repo

```shell
#"[]"中括号一定要存在,里面的名称最好唯一,不然会发生覆盖
[base]
#repository的名字
name=CentOS-$releasever
#表示是否启用,1表示启用,0未启用
enabled=1
#repository的实际地址
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/

[updates]
name=CentOS-$releasever
enabled=1
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
```

#### 命令

```shell
# 安装repository
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```



