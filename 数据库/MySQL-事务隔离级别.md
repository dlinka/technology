**设置隔离级别**

```sql
SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE-READ | SERIALIZABLE};
```

**查看隔离级别**

```sql
SELECT @@GLOBAL.TX_ISOLATION;
SELECT @@SESSION.TX_ISOLATION;
```

**默认隔离级别**

```
REPEATABLE-READ
```

---

#### RU(读取未提交)演示脏读

**客户端A**

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update user set name='zj' where userid=1;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from user;
+--------+------+
| userid | name |
+--------+------+
|      1 | zj   |
+--------+------+
1 row in set (0.00 sec)
```

**客户端B**

```sql
mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
Query OK, 0 rows affected (0.00 sec)

-- 查询结果显示客户端A中未提交事务的数据
mysql> select * from user;
+--------+------+
| userid | name |
+--------+------+
|      1 | zj   |
+--------+------+
1 row in set (0.00 sec)
```

---

#### RC(读取已提交)演示不可重复读

**客户端A**

```sql
mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from user;
+--------+------+
| userid | name |
+--------+------+
|      1 | cr   |
+--------+------+
1 row in set (0.00 sec)
```

**客户端B**

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update user set name='zj' where userid=1;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

**客户端A此时未有脏读现象**

```sql
mysql> select * from user;
+--------+------+
| userid | name |
+--------+------+
|      1 | cr   |
+--------+------+
1 row in set (0.01 sec)
```

**客户端B**

```sql
mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```

**客户端A**

```sql
-- 两次读取不一样
mysql> select * from user;
+--------+------+
| userid | name |
+--------+------+
|      1 | zj   |
+--------+------+
1 row in set (0.00 sec)
```

---

RR(可重复读)演示幻读

**客户端A**

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from user;
+--------+------+
| userid | name |
+--------+------+
|      1 | cr   |
+--------+------+
1 row in set (0.00 sec)
```

**客户端B**

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into user values(2, 'zj');
Query OK, 1 row affected (0.01 sec)
```

**客户端A此时未有脏读现象**

```sql
mysql> select * from user;
+--------+------+
| userid | name |
+--------+------+
|      1 | cr   |
+--------+------+
1 row in set (0.00 sec)
```

**客户端B**

```sql
mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```

**客户端A此时未有不可重复读现象**

```sql
mysql> select * from user;
+--------+------+
| userid | name |
+--------+------+
|      1 | cr   |
+--------+------+
1 row in set (0.00 sec)
```

**客户端A**

```sql
mysql> insert into user values(2, 'zj');
ERROR 1062 (23000): Duplicate entry '2' for key 'PRIMARY'
```

---

