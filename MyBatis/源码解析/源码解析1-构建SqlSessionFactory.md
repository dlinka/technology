## 1.SqlSessionFactoryBuilder#build

```java
public SqlSessionFactory build(InputStream inputStream) {
	return build(inputStream, null, null);
}
↓
↓
public SqlSessionFactory build(InputStream inputStream,
                               String environment,
                               Properties properties) {
	XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
  //解析mybatis-config.xml
  Configuration configuration = parser.parse(); //2
  return build(configuration);
}
↓
↓
public SqlSessionFactory build(Configuration config) {
  return new DefaultSqlSessionFactory(config);
}
```

## 2.XMLConfigBuilder#parse

```java
public Configuration parse() {
	parseConfiguration(parser.evalNode("/configuration"));
  return configuration;
}
↓
↓
private void parseConfiguration(XNode root) {
  mapperElement(root.evalNode("mappers"));
}
↓
↓
private void mapperElement(XNode parent) throws Exception {
  //解析<mapper resource="org/mybatis/example/BlogMapper.xml"/> 
  String resource = child.getStringAttribute("resource");
  if(resource != null && url == null && mapperClass == null) {
    InputStream inputStream = Resources.getResourceAsStream(resource);
    XMLMapperBuilder mapperParser = new XMLMapperBuilder(
      inputStream, configuration, resource, configuration.getSqlFragments()
    );
    //解析BlogMapper.xml
    mapperParser.parse();
  }
}
```

## 3.XMLMapperBuilder#parse

```java
public void parse() {
  configurationElement(parser.evalNode("/mapper")); //4
  bindMapperForNamespace(); //5
}
```

## 4.XMLMapperBuilder#configurationElement

```java
private void configurationElement(XNode context) {
	String namespace = context.getStringAttribute("namespace");
  builderAssistant.setCurrentNamespace(namespace);
  cacheElement(context.evalNode("cache"));
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
  final XMLStatementBuilder statementParser = new XMLStatementBuilder(
    configuration, builderAssistant, context, requiredDatabaseId
  );
  statementParser.parseStatementNode(); //4.1
}
```

## 4.1.XMLStatementBuilder#parseStatementNode

```java
public void parseStatementNode() {
  String id = context.getStringAttribute("id");
  
	builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
        resultSetTypeEnum, flushCache, useCache, resultOrdered,
        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets); //4.2
}
```

## 4.2.MapperBuilderAssistant#addMappedStatement

```java
public MappedStatement addMappedStatement(String id,
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
	//将statement添加到configuration对象的mappedStatements属性中
	configuration.addMappedStatement(statement);
  return statement;
}
```

## 5.XMLMapperBuilder#bindMapperForNamespace

```java
private void bindMapperForNamespace() {
	String namespace = builderAssistant.getCurrentNamespace();
  
  Class<?> boundType = Resources.classForName(namespace);

	if(boundType != null && !configuration.hasMapper(boundType)){
		configuration.addMapper(boundType);
  }
}
```

## 5.1.Configuration#addMapper

```java
public <T> void addMapper(Class<T> type) {
  mapperRegistry.addMapper(type);
}
```

## 5.2.MapperRegistry#addMapper

```java
public <T> void addMapper(Class<T> type) {
	//knownMappers类型定义Map<Class<?>, MapperProxyFactory<?>>
  knownMappers.put(type, new MapperProxyFactory<T>(type));	
}
```

---

