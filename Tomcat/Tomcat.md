#### 启动

```shell
#Start Catalina in the current window
./catalina.sh run

#Start Catalina in a separate window
./catalina.sh start

#Stop Catalina, waiting up to 5 seconds for the process to end
./catalina.sh stop
```



#### 清空catalina.out

```shell
cat /dev/null > catalina.out
```



#### 端口冲突

Tomcat8中的server.xml

```xml
<!-- 修改1 -->
<Server port="8005" shutdown="SHUTDOWN">
<Server port="8006" shutdown="SHUTDOWN">
<!-- 修改2 -->
<Connector port="8027" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
<Connector port="8028" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
<!-- 修改3 -->
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
<Connector port="8010" protocol="AJP/1.3" redirectPort="8443" />
```

