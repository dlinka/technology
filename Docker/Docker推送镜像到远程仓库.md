### Docker Hub

```shell
#登录
docker login -u dlinka
docker tag image:tag dlinka/image:tag
docker push dlinka/image:tag
```

### 阿里云

#### 容器镜像服务

![](/Users/dlinka/GitHub/technology/Docker/aliyun2.png)

#### 选择对应的实例，创建命名空间

![](/Users/dlinka/GitHub/technology/Docker/aliyun3.png)

#### 创建镜像仓库

![](/Users/dlinka/GitHub/technology/Docker/aliyun4.png)

#### 本地仓库

![](/Users/dlinka/GitHub/technology/Docker/aliyun5.png)

#### 使用命令推送

```shell
docker login --username=xxx registry.cn-hangzhou.aliyuncs.com
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/[命名空间]/[仓库名称]:[镜像版本号]
docker push registry.cn-hangzhou.aliyuncs.com/[命名空间]/[仓库名称]:[镜像版本号]
```

---
