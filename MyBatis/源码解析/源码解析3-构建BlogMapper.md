### 官方示例

```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
```

---

### 关键类

#### MapperProxyFactory

```java
public class MapperProxyFactory<T> {
	private final Class<T> mapperInterface;
  private final Map<Method, MapperMethodInvoker> methodCache = new ConcurrentHashMap<>();
}
```

---

### 源码解析

#### 1.进入DefaultSqlSession的getMapper方法

```java
public <T> T getMapper(Class<T> type) {
  return configuration.getMapper(type, this);
}
```

#### 2.Configuration#getMapper

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
  return mapperRegistry.getMapper(type, sqlSession);
}
```

#### 3.MapperRegister#getMapper方法

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
	final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
	return mapperProxyFactory.newInstance(sqlSession);
}
```

#### 4.MapperProxyFactory#newInstance

```java
public T newInstance(SqlSession sqlSession) {
  //MapperProxy实现了动态代理中的InvocationHandler接口
  final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
  return newInstance(mapperProxy);
}
↓
↓
protected T newInstance(MapperProxy<T> mapperProxy) {
  return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
}
```

---

