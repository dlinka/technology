

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
public boolean save(){
  TransactionSynchronizationManagerUtils.invoke(() -> sendMQ());
}

public void sendMQ(){
  someMapper.insert(someEntity);
  producer.sendMessage(message);
}
```

解决

```java
@Transactional
public boolean save(){
  sendMQ());
}

public void sendMQ(){
  someMapper.insert(someEntity);
  TransactionSynchronizationManagerUtils.invoke(() -> producer.sendMessage(message));
}
```

---