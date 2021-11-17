```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
try (SqlSession session = sqlSessionFactory.openSession()) {
  User user = new User("boy", 27);
  mapper.saveUser(user);
  session.commit(); //2
}
```

---

### 事务提交

#### 1.DefaultSqlSession#commit

```java
public void commit() {
  commit(false);
}
↓
↓
public void commit(boolean force) {
  try {
    executor.commit(isCommitOrRollbackRequired(force));
    dirty = false;
  }
}
↓
↓
private boolean isCommitOrRollbackRequired(boolean force) {
	//autoCommit默认是false && 执行saveUser后dirty会变成true
  return (!autoCommit && dirty) || force;
}
```

#### .BaseExecutor#commit

```java
public void commit(boolean required) throws SQLException {
  clearLocalCache();
  flushStatements();
	if (required) {
		transaction.commit();
  }
}
```

#### 3.JdbcTransaction#commit

```java
if (connection != null && !connection.getAutoCommit()) {
	//提交事务
  connection.commit();
}
```

---

### 关闭SqlSession

#### 1.DefaultSqlSession#close

```java
public void close() {
  try {
    //isCommitOrRollbackRequired返回false,事务提交后dirty会重新赋值为false
  	executor.close(isCommitOrRollbackRequired(false));
  }
}
```

#### 2.BaseExecutor#close

```java
public void close(boolean forceRollback) {
	try {
    try {
      rollback(forceRollback);
    } finally {
      if (transaction != null) {
        transaction.close(); //3
      }
    }
  }
}
↓
↓
public void rollback(boolean required) throws SQLException {
	if (!closed) {
  	try {
      clearLocalCache();
      flushStatements(true);
    } finally {
      //required等于false,不执行回滚操作
      if (required) {
        transaction.rollback(); //2
      }
    }
  } 
}
```

#### 2.JdbcTransaction#rollback

```java
public void rollback() throws SQLException {
  if (connection != null && !connection.getAutoCommit()) {
    connection.rollback();
  }
}
```

#### 3.JdbcTransaction#close

```java
public void close() throws SQLException {
  if (connection != null) {
    resetAutoCommit();
  	//关闭连接
    connection.close();
  }
}
```

---