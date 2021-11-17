### 关键类

#### MapperProxy

```java
public class MapperProxy<T> implements InvocationHandler, Serializable {
  private final SqlSession sqlSession;
  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethodInvoker> methodCache;
}
```

#### PlainMethodInvoker

```java
private static class PlainMethodInvoker implements MapperMethodInvoker {
	private final MapperMethod mapperMethod;
}
```

#### MappperMethod

```java
public class MapperMethod {
	private final SqlCommand command;
  private final MethodSignature method;
}
```

#### SqlCommand

```java
public static class SqlCommand {
	//接口.方法名
  private final String name;
  //SQL类型
	private final SqlCommandType type;
}
```

#### MethodSignature

```java
public static class MethodSignature {
  private final Class<?> returnType;
}
```

---

### 源码解析

#### 1.MapperProxy#invoke

**BlogMapper是一个代理对象**

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	MapperMethodInvoker methodInvoker = cachedInvoker(method);
  return methodInvoker.invoke(proxy, method, args, sqlSession); //2
}
↓
↓
private MapperMethodInvoker cachedInvoker(Method method) throws Throwable {
	MapperMethodInvoker invoker = methodCache.get(method);
  if (invoker != null) {
		return invoker;
	}
  return methodCache.computeIfAbsent(method, m -> {
    MapperMethod mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration())
  	return new PlainMethodInvoker(mapperMethod);
  }
}
```

#### 2.PlainMethodInvoker#invoke方法

```java
public Object invoke(Object proxy, Method method, Object[] args, SqlSession sqlSession) throws Throwable {
	return mapperMethod.execute(sqlSession, args); //3
}
```

#### 3.MapperMethod#execute

```java
public Object execute(SqlSession sqlSession, Object[] args) {
	switch (command.getType()) {
	  case SELECT:
  		else {
				Object param = method.convertArgsToSqlCommandParam(args);
    		result = sqlSession.selectOne(command.getName(), param); //4
    	}
  }
}
```

#### 4.DefaultSqlSession#selectOne

```java
public <T> T selectOne(String statement, Object parameter) {
	List<T> list = this.selectList(statement, parameter);
  if (list.size() == 1) {
		return list.get(0);
	}
}
↓
↓
public <E> List<E> selectList(String statement, Object parameter) {
	return this.selectList(statement, parameter, RowBounds.DEFAULT);
}
↓
↓
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
	MappedStatement ms = configuration.getMappedStatement(statement);
  return executor.query(ms, wrapCollection(parameter), rowBounds, null); //5
}
```

#### 5.CachingExecutor#query

```java
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
  BoundSql boundSql = ms.getBoundSql(parameterObject);
  CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
	return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
↓
↓
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  Cache cache = ms.getCache();
  if (cache != null) {
  }
  //CachingExecutor代理SimpleExecutor
	return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql); //6
}
```

#### 6.BaseExecutor#query

**BaseExecutor是SimpleExecutor的父类**

```java
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
}
↓
↓
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
	List<E> list;
  localCache.putObject(key, EXECUTION_PLACEHOLDER);
  try {
  	list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql); //7
  } finally {
  	localCache.removeObject(key);
	}
  localCache.putObject(key, list);
  return list;
}
```

#### 7.SimpleExecutor#doQuery

```java
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
  Statement stmt = null;
  try {
    Configuration configuration = ms.getConfiguration();
    //handler的实现类是RoutingStatementHandler
  	//RoutingStatementHandler代理PreparedStatementHandler
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
    stmt = prepareStatement(handler, ms.getStatementLog());
    return handler.query(stmt, resultHandler); //10
  } finally {
    closeStatement(stmt);
  }
}
↓
↓
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
  Connection connection = getConnection(statementLog); //8
  Statement stmt = handler.prepare(connection, transaction.getTimeout()); //9
  return stmt;
}
```

#### 8.JdbcTransaction#getConnection

```java
public Connection getConnection() throws SQLException {
  if (connection == null) {
    openConnection();
  }
  return connection;
}
↓
↓
protected void openConnection() throws SQLException {
  //从数据源获取连接
	this.connection = dataSource.getConnection();
  //设置事务隔离级别
  if (level != null) {
  	connection.setTransactionIsolation(level.getLevel());
	}
  //设置事务是否自动提交
	setDesiredAutoCommit(autoCommit);
}
```

#### 9.BaseStatementHandler#prepare

```java
public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
	Statement statement = instantiateStatement(connection);
  setStatementTimeout(statement, transactionTimeout);
  return statement;
}
```

#### 9.1PreparedStatementHandler#instantiateStatement

```java
protected Statement instantiateStatement(Connection connection) throws SQLException {
	String sql = boundSql.getSql();
  else if (mappedStatement.getResultSetType() == ResultSetType.DEFAULT) {
    //从Connection获取Statement
  	return connection.prepareStatement(sql);
  }
}
```

#### 10.PreparedStatementHandler#query

```java
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
	PreparedStatement ps = (PreparedStatement) statement;
  ps.execute();
	return resultSetHandler.handleResultSets(ps);
}
```

