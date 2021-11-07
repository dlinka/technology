### 官方示例

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

---

### 关键类

#### Configuration

```java
public class Configuration {
  protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection")
    .conflictMessageProducer((savedValue, targetValue) ->
        ". please check " + savedValue.getResource() + " and " + targetValue.getResource());
	protected final MapperRegistry mapperRegistry = new MapperRegistry(this);
}
```

#### MapperRegistry

```java
public class MapperRegistry {
  private final Configuration config;
	private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();
}
```

---

### 源码解析

#### 1.SqlSessionFactoryBuilder#build方法

```java
public SqlSessionFactory build(InputStream inputStream) {
	return build(inputStream, null, null);
}
↓
↓
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
	XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
  //解析配置文件
  Configuration configuration = parser.parse(); //2
  return build(configuration);
}
↓
↓
public SqlSessionFactory build(Configuration config) {
	//SqlSessionFactory的实现类
  return new DefaultSqlSessionFactory(config);
}
```

#### 2.XMLConfigBuilder#parse

```java
public Configuration parse() {
  parsed = true;
	parseConfiguration(parser.evalNode("/configuration"));
  return configuration;
}
↓
↓
private void parseConfiguration(XNode root) {
  //plugins标签
  pluginElement(root.evalNode("plugins"));
  //environments标签
  environmentsElement(root.evalNode("environments"));
  //mappers标签
  mapperElement(root.evalNode("mappers"));
}
↓
↓
private void mapperElement(XNode parent) throws Exception {
	//mappers标签中的mapper标签
  //<mapper resource="org/mybatis/example/BlogMapper.xml"/> 
  for (XNode child : parent.getChildren()) {
  	else {
      if(resource != null && url == null && mapperClass == null) {
        InputStream inputStream = Resources.getResourceAsStream(resource);
				XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
        //解析Mapper文件
				mapperParser.parse();
			}
    }
  }
}
```

#### 3.XMLMapperBuilder#parse

```java
public void parse() {
	if (!configuration.isResourceLoaded(resource)) {
  	configurationElement(parser.evalNode("/mapper")); //4
    bindMapperForNamespace(); //5
  }
}
```

#### 4.XMLMapperBuilder的configurationElement

```java
private void configurationElement(XNode context) {
  builderAssistant.setCurrentNamespace(namespace);
  buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
}
↓
↓
private void buildStatementFromContext(List<XNode> list) {
	buildStatementFromContext(list, null);
}
↓
↓
private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
	for (XNode context : list) {
    final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
    //select|insert|update|delete标签
    statementParser.parseStatementNode(); //4.1
	}
}
```

#### 4.1.XMLStatementBuilder#parseStatementNode方法

```java
public void parseStatementNode() {
  String id = context.getStringAttribute("id");
  LanguageDriver langDriver = getLanguageDriver(lang);
  SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
	builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
        resultSetTypeEnum, flushCache, useCache, resultOrdered,
        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets); //4.2
}
```

#### 4.2MapperBuilderAssistant#addMappedStatement

```java
public MappedStatement addMappedStatement(
    String id,
    SqlSource sqlSource,
    StatementType statementType,
    SqlCommandType sqlCommandType,
    Integer fetchSize,
    Integer timeout,
    String parameterMap,
    Class<?> parameterType,
    String resultMap,
    Class<?> resultType,
    ResultSetType resultSetType,
    boolean flushCache,
    boolean useCache,
    boolean resultOrdered,
    KeyGenerator keyGenerator,
    String keyProperty,
    String keyColumn,
    String databaseId,
    LanguageDriver lang,
    String resultSets) {
  //构建MappedStatement
  MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
    .resource(resource)
    .fetchSize(fetchSize)
    .timeout(timeout)
    .statementType(statementType)
    .keyGenerator(keyGenerator)
    .keyProperty(keyProperty)
    .keyColumn(keyColumn)
    .databaseId(databaseId)
    .lang(lang)
    .resultOrdered(resultOrdered)
    .resultSets(resultSets)
    .resultMaps(getStatementResultMaps(resultMap, resultType, id))
    .resultSetType(resultSetType)
    .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
    .useCache(valueOrDefault(useCache, isSelect))
    .cache(currentCache);
  MappedStatement statement = statementBuilder.build();
	//MappedStatement对象添加到configuration对象的mappedStatements((Map<String, MappedStatement> mappedStatements))属性中
	configuration.addMappedStatement(statement);
  return statement;
}
```

#### 5.XMLMapperBuilder#bindMapperForNamespace

```java
private void bindMapperForNamespace() {
	String namespace = builderAssistant.getCurrentNamespace();
  if(namespace != null){
    Class<?> boundType = null;
  	boundType = Resources.classForName(namespace);
  }
	if(boundType != null && !configuration.hasMapper(boundType)){
		configuration.addMapper(boundType);
  }
}
```

#### 5.1.Configuration#addMapper

```java
public <T> void addMapper(Class<T> type) {
  mapperRegistry.addMapper(type);
}
```

#### 5.2MapperRegistry#addMapper

```java
//knownMappers类型定义Map<Class<?>, MapperProxyFactory<?>>
knownMappers.put(type, new MapperProxyFactory<T>(type));
```

---

