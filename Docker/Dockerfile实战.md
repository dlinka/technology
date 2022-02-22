```shell
#文件名为Dockerfile,默认可以不用写-f
docker build -t image:tag .
```

---

##### 构建一个diy的centos镜像

Dockerfile

```shell
FROM centos:7
MAINTAINER CR<dlinka.cr7@gmail.com>
ENV JAVA_HOME /usr/local/java
WORKDIR /dlinka

# 安装net-tools
RUN yum -y install vim
RUN yum -y install net-tools

# 这里写了/bin/bash，docker run的时候就不需要写/bin/bash
CMD /bin/bash
```

build

```shell
#.是指定镜像构建过程中的上下文环境的目录为当前目录
docker build -t centos-cr:1.0 .
```

---

##### 构建一个SpringBoot镜像

```shell
FROM java:8
MAINTAINER CR<dlinka.cr7@gmail.com>
WORKDIR /usr/local

COPY demo.jar springboot/
CMD java -jar /usr/local/springboot/demo-0.0.1-SNAPSHOT.jar
```

---