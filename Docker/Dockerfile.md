如何构建一个自己的centos镜像

Dockerfile

    FROM centos
    MAINTAINER CR<dlinka.cr7@gmail.com>
    ENV ENTRY_PATH /cr
    WORKDIR $ENTRY_PATH
    
    # 安装需要的工具
    RUN yum -y install vim
    RUN yum -y install net-tools
    
    # 运行进入bash
    # 这里写了/bin/bash,运行镜像的时候命令就不需要写/bin/bash了
    CMD /bin/bash
    
构建

    docker build -f Dockerfile -t centos-diy:1.0 .

---

如何构建一个自己的tomcat镜像

Dockerfile

    FROM centos
    MAINTAINER CR<dlinka.cr7@gmail.com>

    # 下载下面两个文件跟Dockerfile同目录
    ADD apache-tomcat-9.0.37.tar.gz /usr/local
    ADD jdk-8u221-linux-x64.tar.gz /usr/local
    
    WORKDIR /usr/local
    ENV JAVA_HOME /usr/local/jdk1.8.0_221
    EVN CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    
    CMD /usr/local/apache-tomcat-9.0.37/bin/catalina.sh run

构建

    //文件名为Dockerfile,默认可以不用写-f
    docker build -t tomcat-diy:1.0 .


---

命令手册

CMD

    CMD只有最后一个会生效

ENTRYPOINT

    ENTRYPOING可以追加命令
    Dockerfile:ENTRYPOINT ["ls","-a"]
    执行的时候可以:docker run -it imageId -l
