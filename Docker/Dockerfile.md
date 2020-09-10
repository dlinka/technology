如何构建一个自己的centos镜像

    FROM centos
    MAINTAINER cr<dlinka.cr7@gmail.com>
    ENV ENTRY_PATH /cr
    WORKDIR $ENTRY_PATH
    
    # 安装需要的工具
    RUN yum -y install vim
    RUN yum -y install net-tools
    
    # 运行进入bash
    CMD /bin/bash

---
     
