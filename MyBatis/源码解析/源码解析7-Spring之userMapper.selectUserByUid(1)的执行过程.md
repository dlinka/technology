### 关键类

#### SqlSessionInterceptor

```java 
private class SqlSessionInterceptor implements InvocationHandler {
}
```

---

### 源码解析

#### 1.MapperProxy#invoke

**userMapper是一个代理对象,执行方法会进入MapperProxy的invoke方法**

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  return cachedInvoker(method).invoke(proxy, method, args, sqlSession);
}
↓
↓
//PlainMethodInvoker#invoke
public Object invoke(Object proxy, Method method, Object[] args, SqlSession sqlSession) throws Throwable {
  return mapperMethod.execute(sqlSession, args);
}
↓
↓
//MapperMethod#execute
public Object execute(SqlSession sqlSession, Object[] args) {
	Object param = method.convertArgsToSqlCommandParam(args);
  //此处跟之前唯一的不同就是sqlSession的实现类,这里是SqlSessionTemplate
	result = sqlSession.selectOne(command.getName(), param); //2
}
```

#### 2.SqlSessionTemplate#selectOne

```java
public <T> T selectOne(String statement, Object parameter) {
  return this.sqlSessionProxy.selectOne(statement, parameter); //3
}
```

#### 3.SqlSessionInterceptor#invoke

**sqlSessionProxy也是一个代理对象,执行方法会进入SqlSessionInterceptor#invoke**

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	SqlSession sqlSession = getSqlSession(
    			SqlSessionTemplate.this.sqlSessionFactory,
          SqlSessionTemplate.this.executorType,
    			SqlSessionTemplate.this.exceptionTranslator); //4
  try {
  	Object result = method.invoke(sqlSession, args);
    return result;
  } finally {
    if (sqlSession != null) {
  		closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
		}
  }
}
```

#### 4.SqlSessionUtils#getSqlSession

```java
public static SqlSession getSqlSession(SqlSessionFactory sessionFactory, ExecutorType executorType,
      PersistenceExceptionTranslator exceptionTranslator) {
	SqlSessionHolder holder = (SqlSessionHolder) TransactionSynchronizationManager.getResource(sessionFactory);
  SqlSession session = sessionHolder(executorType, holder);
  if (session != null) {
		return session;
	}
  //跟之前从SqlSessionFactory中获取SqlSession一样的逻辑
  session = sessionFactory.openSession(executorType);
  registerSessionHolder(sessionFactory, executorType, exceptionTranslator, session);
  return session;
}
```

---
