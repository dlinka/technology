#### count

```
查询表数据量很大的时候count会很慢，可以考虑另建一个表来专门存储统计数据，比如可以存放表的名称和对应的count数
```



---

#### 索引

```
阿里巴巴开发者手册规定,单表的索引数量尽量控制在5个以内,并且单个索引的字段数不超过5个
```

force index(XXX)

```
一条sql只会用到一个索引,MySQL优化器会计算出一个合适的索引,但是这个索引不一定是最好的
所以可以使用force index强制使用某个高效索引
```



---



#### join

**代替子查询**

```sql
-- 子查询语句使用in关键字实现,会先运行嵌套内层的语句
-- MySQL执行子查询时,需要创建临时表,查询完毕后,再删除这些临时表
select * from order where user_id in (select id from user where status=1)

-- 使用inner join替代
select o.* from order o
inner join user u on o.user =  u.id
where u.status = 1
```

**控制join表的数量**

```
阿里巴巴开发者手册规定,join表的数量不应该超过3个

现实业务中如果需要查询出另外几张表的数据,可以在另外几张表中冗余专门的字段

还是需要根据系统的实际情况决定,不能一概而论,反正尽量越少越好
```

**left join、inner join**

```
使用inner join,MySQL会自动选择两张表中的小表,去驱动大表

使用left join,MySQL用left join左边的表去驱动右边的表,如果左边的表数据很多的时候,就会出现性能问题
```





---

#### 多用limit

```
删除或者更新数据时,为了防止误操作,导致删除或者修改了不相干的数据,也可以在SQL语句上加上limit
这样即使误操作,也不会对太多的数据造成影响
```



---

#### 小表驱动大表

order和user两张表

分别使用in和exists查询所有有效用户下过的订单列表

```sql
-- SQL中含有in关键字,会优先执行in的子查询语句
-- 如果order的数据大于user的数据,这样的性能比下面exist方式更好
select * from order where user_id in (select id from user where status=1)

-- SQL中含有exist关键字,会优先执行exits左边的语句
-- 如果order的数据小于user的数据,这样的性能比上面in方式更好
select * from order where exists (select 1 from user where order.user_id = user.id and status=1)
```



---

#### 避免select *

```
SQL查询的时候,只查需要用到的列,多余的列不要查出来
因为select *会出现回表查询,而且会增加数据传输时间
```



---

如何查询是否存在

    生产环境有个10万行的表fitness_session
    
    SQL
    select count(*) from fitness_session where device_id=10000058 and status=2
    执行时长平均50ms,结果3644,表示所有满足条件的数量
    
    优化方案
    select 1 from fitness_session where device_id=10000058 and status=2 limit 1
    执行时长平均25ms,结果就是1
    
    让数据库查询时遇到1条就返回,不用继续查询还有多少条了

---

更新大量数据

    更新所有已过期优惠券
    如果行数量很大,这种SQL可能会导致MySQL卡死
    update status=2 from coupon WHERE expire_date <= now()
    
    解决方案:
    分批处理,for循环,1次更新100条

---

or不会使用索引

    i1和d1是一个组合索引
    此时并不会使用索引(type=ALL,Extra=Using where)
    select * from table where i1=1 or d1=1
    
    可以分别在i1和d1上建立索引
    然后使用如下SQL优化方案
    (select * from table where i1=1)
    union
    (select * from table where d1=1)

#### 分页查询

分页查询越到后面时间越长

```sql
-- 这种方式会查询1000020条数据,然后丢弃前面的1000000,只留后面的20条数据,非常浪费资源
select id,name,age from user limit 1000000,20
↓
↓
-- 优化1
-- 找到分页最大的id,然后利用id上的索引查询
select id,name,age from user where id > 1000000 limit 20
-- 优化2
-- between区间查询
select id,name,age from user where id between 1000000 and 1000020
```

---

利用位计算

    UPDATE table SET column=column|4
    27|4
    ↓
    0001 1011|0000 0100
    ↓
    0001 1111
    ↓
    31
    
    UPDATE table SET column=column|8 WHERE column&1=1
    3847|8
    ↓
    1111 0000 0111|0000 0000 1000
    ↓
    1111 0000 1111
    ↓
    3855
    
    3847&1
    ↓
    1111 0000 0111&0000 0000 0001
    ↓
    0000 0000 0001
    ↓
    1

---



#### 提升效率

```sql
-- 这种写法会把所有的订单根据用户id分组,再去过滤用户id大于等于200的用户
select user_id, user_name from order
group by user_id
having user_id <= 200
↓
↓
-- 先缩小数据范围,再分组
select user_id,user_name from order
where user_id <= 200
group by user_id
```





---



