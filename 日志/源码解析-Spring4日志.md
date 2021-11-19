```
尝试加载的那几个类就是自己jar中的，那为什么加载自己jar包中的类会出现不成功呢？原因就在于jcl这个包中有关于log4j的optional=true的依赖
```

### Maven依赖

```java
[INFO] +- org.springframework:spring-context:jar:4.3.5.RELEASE:compile
[INFO] |  +- org.springframework:spring-core:jar:4.3.5.RELEASE:compile
[INFO] |  |  \- commons-logging:commons-logging:jar:1.2:compile
```

#### 1.LogFactory#getLog

```java
//org.apache.commons.logging.LogFactory这个类在commons-logging包中
return getFactory().getInstance(clazz);
```

#### 2.LogFactoryImpl#getInstance

```java
//org.apache.commons.logging.impl.LogFactoryImpl
return getInstance(clazz.getName());
↓
↓
Log instance = (Log) instances.get(name);
if (instance == null) {
    //创建Log
    instance = newInstance(name);
    instances.put(name, instance);
}
return instance;
↓
↓
if (logConstructor == null) {
	instance = discoverLogImplementation(name);
}
↓
↓
//按照如下顺序加载,找到就不在找了
//org.apache.commons.logging.impl.Log4JLogger(log4j)
//org.apache.commons.logging.impl.Jdk14Logger(jul)
//org.apache.commons.logging.impl.Jdk13LumberjackLogger
//org.apache.commons.logging.impl.SimpleLog
for(int i=0; i<classesToDiscover.length && result == null; ++i) {
    result = createLogFromClass(classesToDiscover[i], logCategory, true);
}
return result;
```

---

