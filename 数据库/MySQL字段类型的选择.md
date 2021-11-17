时间类型

    timestamp占用4个字节
    datetime占用8个字节
    
    假设我的一个项目有十个表,每个表都有create_time,update_time,每个表最少一千万的数据
    使用timestamp会比使用datetime剩下多少空间的大小
    4*2(个字段)*10(个表)*10000000=8亿个字节
    8亿个字节约等于800MB

---

#### DECIMAL(M,D)

**MySQL(8.0.24)**

**M表示总位数,D表示小数位数***

**M>=D,0<=M<=65,0<=D<=30**

****

```sql
create table test(
    decimal_field decimal(10,2) -- 如果不指定(M,D),默认是(10, 0)
)
-- 插入10位整数报错
-- Out of range value for column 'decimal_field' at row 1
insert into test(decimal_field) values(1234567890);

-- 插入8位整数成功
-- 数据库查询结果12345678.00(小数部分会补2个0)
insert into test(id,decimal_field) values(1, 12345678);

-- 插入2位整数,1位小数成功
-- 数据库查询结果12.10(小数部分会补1个0)
insert into test(decimal_field) values(12.1);

-- 插入8位整数,5位小数成功
-- 数据库查询结果12345678.12,发生了截断
insert into test(decimal_field) values(12345678.12345);

-- 插入5位小数成功
-- 数据库查询结果0.13,发生了截断,并四舍五入
insert into test(decimal_field) values(0.12543);
```

    1~2位,用1个字节
    3~4位,用2个字节
    5~6位,用3个字节
    7~9位,用4个字节
    ...
    
    根据上面规则计算DECIMAL(20,6)的占用空间
    小数部分6位数字,整数部分14位数字
    小数部分6位根据上面规则需要3个字节
    整数部分9位需要4个字节,其余5位需要3个字节
    上面合起来就是10个字节

---
