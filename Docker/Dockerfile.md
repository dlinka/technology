```
每条指令都必须为大写字母且后面要跟随至少一个参数
每条指令都会创建一个新的镜像层并提交

docker从基础镜像运行，执行一条指令对容器做出了修改，都会提交一个新的镜像层。docker再基于刚提交的镜像运行一个新的容器，直到所有指令执行完成。

Docker Hub中99%的镜像都是基于scratch构建出来的
```

```shell
FROM #基础镜像
MAINTAINER #维护者
RUN #指定运行命令
EXPOSE
EXPOSE指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务。
在Dockerfile中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射，另一个用处则是在运行时使用随机端口映射时，也就是docker run -P时，会自动随机映射 EXPOSE 的端口。
WORKDIR #工作目录
ENV #环境变量
ADD #文件拷贝到容器中
COPY #文件和目录拷贝到容器中
VOLUME #数据卷
CMD #指定一个容器启动时要运行的命令，如果有多个CMD,只有最后一个会生效，CMD会被docker run之后的参数替换
ENTRYPOINT #docker run之后的参数会被当做参数传递给ENTRYPOINT，之后形成新的命令组合
Dockerfile中定义ENTRYPOINT ["ls","-a"]，那么执行的时候就可以使用docker run -it image -l追加-l参数，而换成CMD的话会被替换为-l。
ONBUILD
```

---
