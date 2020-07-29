**日志**

```java
//控制台grep "[Batch Refresh Task]"这个功能的所有日志
log.info("[Batch Refresh Task] xxx xxx ...", xxx, xxx, ...);
```

---

**扩展性**

```java
//这种设计不能很好的保证扩展性,因为在2-4之间只能插入一个类型3的禁售类型
NORMAL(0)
  SINGLE_DISABLED(2)
  	CATEGORY_DISABLED(4)
  		...
//最好是这样的是下面这种,区间不要太小,如果在1000-2000区间新增加一个类型,可以选择1100或者1500
NORMAL(0)
  SINGLE_DISABLED(1000)
  	CATEGORY_DISABLED(2000)
  
//商品属性必填禁用
MANDATORY_PROPERTY_DISABLE(18, "商品属性变更为必填")
//商品属性非必填解禁(目前不需要这个功能,但是防止占用,预留这个type)
MANDATORY_PROPERTY_ENABLE(19, "商品属性变更为非必填")
```

---

**统计错误日志命令**

```shell
#uniq表示去重复
#	-c	重复出现的次数
#sort表示排序
#	-n	数字大小排序
#	-r	逆序
cat itemdetail-biz.2021-07-25.*|grep "|ERROR"|awk 'BEGIN {FS="|"} {print $10}'|sort|uniq -c|sort -nr > summary_error.log
```

---

##### Demo

```java
@Test
public void testMethod() {
    method();
}
```

```java
@Transactional(rollbackFor = Exception.class)
public void method() {
    TransactionSynchronizationManagerUtils.invoke(() ->
                                                  new Thread(() -> business()));
}
↓
↓
redisTemplate.opsForValue().setIfAbsent(key, value, DEFAULT_TTL_MINUTES, MINUTES);
```

![image](https://user-images.githubusercontent.com/4274041/100562395-9189c180-32f6-11eb-90f6-fb118a6f4c96.png)

解决

```java
@Test
public void testMethod() {
    method();
    try {
        TimeUnit.SECONDS.sleep(600);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

---

##### Demo

```java
@Transactional
public boolean save() {
  TransactionSynchronizationManagerUtils.invoke(() -> sendMQ());
}

public void sendMQ() {
  someMapper.insert(someEntity);
  producer.sendMessage(message);
}
```

解决

```java
@Transactional
public boolean save() {
  sendMQ());
}

public void sendMQ() {
  someMapper.insert(someEntity);
  TransactionSynchronizationManagerUtils.invoke(() -> producer.sendMessage(message));
}
```

---