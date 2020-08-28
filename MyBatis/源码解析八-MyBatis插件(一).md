官方示例

    <plugins>
        <plugin interceptor="org.mybatis.example.ExamplePlugin">
            <property name="someProperty" value="100"/>
        </plugin>
    </plugins>


1.进入XMLConfigBuilder的parseConfiguration方法

    pluginElement(root.evalNode("plugins"));
    ↓
    ↓
    for (XNode child : parent.getChildren()) {
        //拿到org.mybatis.example.ExamplePlugin字符串
        String interceptor = child.getStringAttribute("interceptor");
        Properties properties = child.getChildrenAsProperties();
        Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
        interceptorInstance.setProperties(properties);
        //将拦截器对象添加到configuration的interceptorChain属性中
        configuration.addInterceptor(interceptorInstance);
    }

2.插件使用的场景

    Configuration的newExecutor方法
    executor = (Executor) interceptorChain.pluginAll(executor);
    
    SimpleExecutor的doQuery方法
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
    ↓
    ↓
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    
    PreparedStatementHandler的父类构造函数BaseStatementHandler
    this.parameterHandler = configuration.newParameterHandler(mappedStatement, parameterObject, boundSql);
    this.resultSetHandler = configuration.newResultSetHandler(executor, mappedStatement, rowBounds, parameterHandler, resultHandler, boundSql);
    ↓
    ↓
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler); 
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    
3.进入InterceptorChain的pluginAll()方法

    //遍历拦截器
    for (Interceptor interceptor : interceptors) {
      //执行拦截器的plugin方法
      target = interceptor.plugin(target);
    }
    return target;
    
---
