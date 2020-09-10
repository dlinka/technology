如何构建一个自己的centos镜像

Dockerfile

    FROM centos
    MAINTAINER cr<dlinka.cr7@gmail.com>
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
