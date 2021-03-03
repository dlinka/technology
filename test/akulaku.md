

##### Demo

问题

```java
@Transactional(rollbackFor = Exception.class)
public void method(Message message) {
    TransactionSynchronizationManagerUtils.invoke(() -> threadPool.execute(() -> afterMessageSave()));
}
↓
↓
//afterMessageSave
Boolean lock = redisTemplate.opsForValue().setIfAbsent(lockKey, lockValue, DEFAULT_TTL_MINUTES, MINUTES);
```

```java
@Test
public void testMethod() {
    method(message);
}
```

![image](https://user-images.githubusercontent.com/4274041/100562395-9189c180-32f6-11eb-90f6-fb118a6f4c96.png)

解决

```java
@Test
public void testMethod() {
    method(message);
    try {
        TimeUnit.SECONDS.sleep(600);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

---

##### Demo

问题

```java
@Transactional
public boolean saveIllegalWord(){
  TransactionSynchronizationManagerUtils.invoke(() -> sendMQ(entity));
}

public void sendMQ(IllegalWordEntity entity){
  IllegalWordScanTaskEntity taskEntity = new IllegalWordScanTaskEntity();
  taskEntity.setIllegalWordId(entity.getId());
  illegalWordScanTaskMapper.insert(taskEntity);
  ...
  producer.sendMessage(message);
}
```

解决

```java
@Transactional
public boolean saveIllegalWord(){
  sendMQ(entity));
}

public void sendMQ(IllegalWordEntity entity){
  IllegalWordScanTaskEntity taskEntity = new IllegalWordScanTaskEntity();
  taskEntity.setIllegalWordId(entity.getId());
  illegalWordScanTaskMapper.insert(taskEntity);
  ...
  TransactionSynchronizationManagerUtils.invoke(() -> producer.sendMessage(message));
}
```

---