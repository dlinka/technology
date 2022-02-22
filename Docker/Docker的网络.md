#### docker0

宿主机docker0

```shell
ip addr
↓
↓
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:ba:12:20:c5 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
```

---

#### 容器和宿主机之间网络

```shell
docker run -it centos /bin/bash

#容器ip=172.17.0.2
ip addr
↓
↓
268: eth0@if269: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0

#容器ping宿主机
ping 172.17.0.1
↓
↓
PING 172.17.0.1 (172.17.0.1) 56(84) bytes of data.
64 bytes from 172.17.0.1: icmp_seq=1 ttl=64 time=0.045 ms
64 bytes from 172.17.0.1: icmp_seq=2 ttl=64 time=0.058 ms
64 bytes from 172.17.0.1: icmp_seq=3 ttl=64 time=0.063 ms

#退出
ctrl+p+q

#宿主机ping容器
ping 172.17.0.2
↓
↓
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.059 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.055 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.057 ms

#宿主机
ip addr
↓
↓
269: veth316ca32@if268: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP 
    link/ether ce:fd:b7:61:9a:1d brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

#### 容器和容器之间网络

```shell
docker run -it centos /bin/bash

ip addr
↓
↓
270: eth0@if271: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0

#互相ping通
ping 172.17.0.2
↓
↓
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.073 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.033 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.062 ms
```

---

#### --link

```shell
#将container的ip写入hosts
docker run -it --name centos --link container centos:7
```

---

#### Docker自定义网络

##### network

```shell
#显示所有网络
docker network ls
↓
↓
NETWORK ID     NAME      DRIVER    SCOPE
17cc4ca777b1   bridge    bridge    local #默认我们不配置网络，也就相当于默认值--net bridge

#显示网络详细信息
docker network inspect 17cc4ca777b1
↓
↓
[
    {
        "Name": "bridge",
        "Id": "17cc4ca777b1f259ca16c9ba5f07ed62afc3684153165e381a7ab3d656424f9e",
        "Created": "2022-02-19T04:06:04.581843917Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16", #docker0可以创建255*254个ip
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "a52cb2e2b601f5eb515fb91c67c0ea7ab69233fe3d645d0f7507c2b5f5d496f9": {
                "Name": "determined_morse",
                "EndpointID": "b5afd13492a169c48752460fd09b75aee4fdeb8d0630434bfe748996efbaa17d",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

#自定义一个网络
#自定义网络和docker0不通
docker network create --driver bridge  --subnet 192.168.0.0/16 --gateway 192.168.0.1 demo-net

docker run -d -P --net demo-net --name tomcat1 tomcat #启动容器的时候指定网络
docker exec -it tomcat ping 192.168.0.2 #可以ping
docker exec -it tomcat ping tomcat1 #可以ping容器名字

#将名为mysql的容器加入到demo-net网络中
docker network connect demo-net mysql
```

---
