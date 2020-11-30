背景

    @Transactional(rollbackFor = Exception.class, isolation = Isolation.READ_COMMITTED)
    public void method(Message message) {
        //业务逻辑
        ...
        //事务提交后
        //开启线程执行afterMessageSave方法
        TransactionSynchronizationManagerUtils.invoke(() -> THREAD_POOL.execute(() -> afterMessageSave()));
    }
    ↓
    ↓
    //afterMessageSave
    //分布式锁
    Boolean absent = redisTemplate.opsForValue().setIfAbsent(lockKey, lockValue, DEFAULT_TTL_MINUTES, MINUTES);

![image](https://user-images.githubusercontent.com/4274041/100562395-9189c180-32f6-11eb-90f6-fb118a6f4c96.png)

解决

    @Test
    public void testMethod() {
        //业务逻辑
        ...
        XXX.method(message);
        try {
            TimeUnit.SECONDS.sleep(600);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

---
