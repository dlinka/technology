### 官方示例

#### xml

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
</bean>
```

---

### 关键类

#### SqlSessionFactoryBean

```java
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {
	private Configuration configuration;
  private Resource[] mapperLocations;
  private DataSource dataSource;
  private TransactionFactory transactionFactory;
  private Properties configurationProperties;
}
```

---

### 源码解析

#### 1.SqlSessionFactoryBean#afterPropertiesSet

**SqlSessionFactoryBean实现了InitializingBean**

```java
public void afterPropertiesSet() throws Exception {
	this.sqlSessionFactory = buildSqlSessionFactory();	
}
↓
↓
protected SqlSessionFactory buildSqlSessionFactory() throws Exception {
	//跟之前一样,都是先初始化Configuration对象
  final Configuration targetConfiguration;
  XMLConfigBuilder xmlConfigBuilder = null;
  else {
    targetConfiguration = new Configuration();
  }
  //插件
  if (!isEmpty(this.plugins)) {...}
  //Environment
  targetConfiguration.setEnvironment(new Environment(this.environment,
        this.transactionFactory == null ? new SpringManagedTransactionFactory() : this.transactionFactory,
        this.dataSource));
  //跟之前一样,生成DefaultSqlSessionFactory对象做为返回值
  return this.sqlSessionFactoryBuilder.build(targetConfiguration);
}
```

#### 2.SqlSessionFactoryBean#getObject

**SqlSessionFactoryBean实现了FactoryBean**

```java
public SqlSessionFactory getObject() throws Exception {
	if (this.sqlSessionFactory == null) {
  	this.afterPropertiesSet();
  }
  //直接返回上一步生成的DefaultSqlSessionFactory
  return this.sqlSessionFactory;
}
```

---
