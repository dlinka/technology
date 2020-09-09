version 显示Docker版本  
info 显示详细信息,还有容器和镜像的数量  
cmd --help 显示命令用法  

    //显示ps命令用法
    docker ps --help

images 显示镜像  
search 搜索镜像  
pull 下载镜像  
rmi 删除镜像  
run 运行镜像

    //启动centos镜像,并进入容器
    docker run -it centos /bin/bash
    
    //-d表示后台启动
    //centos启动后容器就会停止,因为没有容器内没有进程运行,反之如果有进程运行,容器就不会停止
    docker run -d centos
    
    //-p端口映射
    docker run -d -p 80:80 nginx
    
    //-v挂载
    //宿主机和容器可以同步文件
    docker run -it -v ~/host/centos:~/container/centos centos /bin/bash
    //匿名挂载,只写了容器路径,挂载到的宿主机目录可以通过volume命令查看
    docker run -it -v /container/centos centos /bin/bash
    //具名挂载,前面可以加个volumeName,挂载到的宿主机目录可以通过volume命令查看
    docker run -it -v volumeName:/container/centos centos /bin/bash
    //ro表示readonly
    docker run -it -v /container/centos:ro centos /bin/bash
    //挂载继承
    docker run -it --name centos02 --volumes-from centos01 centos

ps 显示容器

    docker ps -aq //只显示所有容器ID

rm 删除容器  
start 启动容器  
restart 重启容器  
stop 停止容器  
kill 强制停止容器  
exit 退出容器,如果有进程运行容器不会停止  
ctrl+p+q 退出容器,如果没有进程运行容器也不停止  
top 显示容器中进程信息  
inspect 显示容器元数据  
exec 进入容器,开启新的终端  

    docker exec -it 容器ID /bin/bash
    
attach 进入容器.不开启新的终端  
cp 拷贝容器内文件到外面  
stats 查看所有容器使用的资源情况  
commit 基于运行的容器生成一个新镜像

    docker commit -a="cr" -m="message" 66f596225108 tomcat:1.1

volume 查看挂载情况

    //只会显示具名挂载和匿名挂载
    docker volume ls
    
    //显示volumeName的挂载情况
    docker volume inspect volumeName

---
