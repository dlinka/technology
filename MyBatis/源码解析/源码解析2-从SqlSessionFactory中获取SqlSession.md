### 官方示例

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
}
```

---

### 关键类

#### DefaultSqlSession

```java
public class DefaultSqlSession implements SqlSession {
	private final Configuration configuration;
  private final Executor executor;
  private final boolean autoCommit;
}
```

---

### 源码解析

#### 1.DefaultSqlSessionFactory#openSession

```java
public SqlSession openSession() {
  return openSessionFromDataSource(ExecutorType.SIMPLE, null, false);
}
↓
↓
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
	Transaction tx = null;
  final Environment environment = configuration.getEnvironment();
  //返回JdbcTransactionFactory(<transactionManager type="JDBC"/>)
  final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
	//new JdbcTransaction(ds, level, autoCommit)
  tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
  //new SimpleExecutor(this, transaction);
  //new CachingExecutor(executor);
  final Executor executor = configuration.newExecutor(tx, execType);
  //创建DefaultSqlSession
  return new DefaultSqlSession(configuration, executor, autoCommit);
}
```

---

