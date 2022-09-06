## 1.DefaultSqlSessionFactory#openSession

```java
public SqlSession openSession() {
  return openSessionFromDataSource(
    ExecutorType.SIMPLE,
    null,
    false);
}
↓
↓
private SqlSession openSessionFromDataSource(ExecutorType execType, 
                                             TransactionIsolationLevel level,
                                             boolean autoCommit) {
	Transaction tx = null;
  final Environment environment = configuration.getEnvironment();

  //JdbcTransactionFactory(<transactionManager type="JDBC"/>)
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

