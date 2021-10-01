### elasticsearch-head

#### 1.下载地址:https://github.com/mobz/elasticsearch-head

需要node.js环境

![elasticsearch-head-url](/Users/dlinka/GitHub/technology/ELK/elasticsearch-head-url.png)

#### 2.git clone git://github.com/mobz/elasticsearch-head.git

```
cd elasticsearch-head
npm install
```

![elasticserch-head-file](/Users/dlinka/GitHub/technology/ELK/elasticsearch-head-file.png)

#### 3.启动

```
npm run start
```

![elasticserch-head-start](/Users/dlinka/GitHub/technology/ELK/elasticsearch-head-start.png)

#### 4.访问9100

![elasticserch-head-9100](/Users/dlinka/GitHub/technology/ELK/elasticsearch-head-9100.png)



---

#### elasticsearch.yml

```yaml
#跨域
http.cors.enabled: true
http.cors.allow-origin: "*"
```

