

---



### DISTINCT

| id   | name | age  |
| ---- | ---- | ---- |
| 1    | cr   | 18   |
| 2    | CR   | 30   |
| 3    | 陈润 | 18   |

```sql
-- 查询表中不重复的age数量
SELECT COUNT(DISTINCT age) FROM user

-- 查询表中不同的age值
SELECT DISTINCT age FROM user

-- 查询表中name和age都不相同的值
SELECT DISTINCT name, age FROM user
```

---



### DML

#### INSERT、REPLEASE INTO、INSERT IGNORE的区别

|               | 已存在 | 不存在 | SQL                                                    |
| ------------- | ------ | ------ | ------------------------------------------------------ |
| INSERT INTO   | 报错   | 插入   | INSERT INTO user(name, age) VALUES(“小明”, 23);        |
| INSERT IGNORE | 忽略   | 插入   | INSERT IGNORE INTO user(name, age) VALUES(“小明”, 23); |
| REPLEASE      | 替换   | 插入   | REPLEASE INTO user(name, age) VALUES(“小明”, 23);      |

```sql
CREATE TABLE user (
    id INT(10) PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) UNIQUE,
    age INT(10)
)
INSERT INTO user(name, age) VALUES("小明", 24);
INSERT INTO user(name, age) VALUES("小红", 24);
```

##### INSERT

如果已存在,会报错,但是id会增加

```sql
-- 1062 - Duplicate entry '小明' for key 'user.name', Time: 0.002000s
INSERT INTO user(name, age) VALUES("小明", 23);
```

##### REPLEASE INTO

如果已存在,删除原来的记录,添加新的记录

```sql
-- Affected rows: 2, Time: 0.002000s
REPLEASE INTO user(name, age) VALUES("小明", 23);
```

##### INSERT IGNORE

如果已存在,忽略新插入的记录,也就是不会报错,但是id会自增

```sql
-- Affected rows: 0, Time: 0.002000s
INSERT IGNORE INTO user(name, age) values("小明", 23);
```

---

#### ON DUPLICATE KEY UPDATE

```sql
-- 只对id是主键或者唯一索引生效
INSERT INTO user (id,name,age) VALUES (1,'CR',1) ON DUPLICATE KEY UPDATE age=age+1
```

---



### DDL

#### 查看表结构

```sql
DESC table_name
SHOW CREATE TABLE table_name
```



#### ALTER

##### 更新类型、默认值等信息

```sql
-- ALTER TABLE 表名字 MODIFY 字段 新的数据类型 新的默认值 新的注释
ALTER TABLE user MODIFY name VARCHAR(255) NOT NULL COMMENT '名字'
```

##### 更新字段名

```sql
-- ALTER TABLE 表名字 CHANGE 旧字段 新字段 新的数据类型 新的默认值 新的注释
ALTER TABLE user CHANGE name nickname VARCHAR(255) NOT NULL COMMENT '昵称'
```

##### 增加字段

```sql
ALTER TABLE user ADD gender TINYINT(4) DEFAULT '0' COMMENT '性别' AFTER name
```

##### 删除字段

```sql
ALTER TABLE user DROP profile
```



---



### 函数

##### IFNULL

```sql
-- 这行SQL查不出来real_name等于NULL数据
SELECT * FROM user WHERE age!=0
↓
↓
SELECT * FROM user WHERE IFNULL(age, -1)!=0
```

##### UNIX_TIMESTAMP

```sql
-- 这种方式不生效,无论time类型是DATETIME,还是TIMESTAMP
SELECT * FROM table WHERE time > 1580970685000

SELECT * FROM table WHERE UNIX_TIMESTAMP(time) >= UNIX_TIMESTAMP('2021-06-01 06:01:00')
```

##### DATE_ADD

```sql
-- 加7天
UPDATE table SET column = DATE_ADD(column, INTERVAL 7 DAY)
```

##### DATE_SUB

```sql
-- 减1个月
UPDATE table SET column = DATE_SUB(column, INTERVAL 1 MONTH)
```



---
