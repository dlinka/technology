##### 拉取镜像

```shell
docker pull centos/mysql-57-centos7
```

##### 启动镜像

```shell
docker run -p 33061:3306 --name master-mysql -e MYSQL_ROOT_PASSWORD=123456 -d centos/mysql-57-centos7
docker run -p 33062:3306 --name slave-mysql -e MYSQL_ROOT_PASSWORD=123456 -d centos/mysql-57-centos7
```

##### 配置Master

```shell
#root用户进入容器
docker exec -it --user root master-mysql /bin/bash

vi /etc/my.cnf
[mysqld]
#同一局域网内注意要唯一
server-id=100
#开启二进制日志功能，并且文件
log-bin=master-log-bin

#重启容器
docker restart master-mysql

#进入mysql终端执行SQL
CREATE USER 'docker'@'%' IDENTIFIED BY '1234567890';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'docker'@'%';
```

##### 配置Slave

```shell
docker exec -it --user root slave-mysql /bin/bash

vi /etc/my.cnf
[mysqld]
server-id=101  
log-bin=slave-log-bin
relay-log=slave-relay-bin
```

##### 主从配置

查看master状态

```sql
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      769 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.02 sec)
```

slave连接master

```sql
change master to master_host='172.17.0.2', master_user='docker', master_password='1234567890', master_port=3306, master_log_file='mysql-bin.000002', master_log_pos=769, master_connect_retry=30;
```

查看slave状态

```sql
-- 正常情况下，SlaveIORunning和SlaveSQLRunning显示都是NO，因为我们还没有开启主从复制过程
show slave status;

-- 开启
start slave;

-- 再次查看SlaveIORunning和SlaveSQLRunning显示都是YES
show slave status;
```

