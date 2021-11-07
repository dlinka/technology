### 官方示例

```java
public interface UserMapper {
  @Select("SELECT * FROM users WHERE id = #{userId}")
  User getUser(@Param("userId") String userId);
}
```

#### xml

```xml
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

---

### 关键类

#### MapperFactoryBean

```java
public class MapperFactoryBean<T> extends SqlSessionDaoSupport implements FactoryBean<T> {
	private Class<T> mapperInterface;
}
```

#### SqlSessionDaoSupport

```java
public abstract class SqlSessionDaoSupport extends DaoSupport {
	private SqlSessionTemplate sqlSessionTemplate;
}
```

#### SqlSessionTemplate

```java
public class SqlSessionTemplate implements SqlSession, DisposableBean {
	private final SqlSessionFactory sqlSessionFactory;
  private final ExecutorType executorType;
  private final SqlSession sqlSessionProxy;
}
```

---

### 源码解析

#### 1.MapperFactoryBean的父类SqlSessionDaoSupport#setSqlSessionFactory()


```java
public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
  if (this.sqlSessionTemplate == null || sqlSessionFactory != this.sqlSessionTemplate.getSqlSessionFactory()) {
	  this.sqlSessionTemplate = createSqlSessionTemplate(sqlSessionFactory);
  }
}
↓
↓
protected SqlSessionTemplate createSqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
	//SqlSession在之前的源码解析中实现类是DefaultSqlSession,这里的实现类是SqlSessionTemplate
  return new SqlSessionTemplate(sqlSessionFactory); //2
}
```

#### 2.SqlSessionTemplate的构造方法

```java
public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
  this(sqlSessionFactory, sqlSessionFactory.getConfiguration().getDefaultExecutorType());
}
↓
↓
public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType) {
  this(sqlSessionFactory, executorType,
      new MyBatisExceptionTranslator(sqlSessionFactory.getConfiguration().getEnvironment().getDataSource(), true));
}
↓
↓
public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType,
      PersistenceExceptionTranslator exceptionTranslator) {
	this.sqlSessionFactory = sqlSessionFactory;
  this.executorType = executorType;
  this.sqlSessionProxy = (SqlSession) newProxyInstance(SqlSessionFactory.class.getClassLoader(),
        new Class[] { SqlSession.class }, new SqlSessionInterceptor());
}
```

#### 3.MapperFactoryBean#getObject

**MapperFactoryBean实现了FactoryBean接口**

```java
public T getObject() throws Exception {
	//getSqlSession()方法返回上一步创建的SqlSessionTemplate对象
  return getSqlSession().getMapper(this.mapperInterface); //4
}
```

#### 4.SqlSessionTemplate#getMapper()方法

```java
public <T> T getMapper(Class<T> type) {
  //之前源码解析生成的动态代理对象传入的是DefaultSqlSession
	//现在这里生成的动态代理对象传入的是SqlSessionTemplate
  return getConfiguration().getMapper(type, this);
}
```

---

**SqlSessionTemplate的官方API的一段描述,这个对象可以单例存在**

![image](https://user-images.githubusercontent.com/4274041/81561452-c7507280-93c5-11ea-8aa5-4f8e7c9b26bf.png)

---
