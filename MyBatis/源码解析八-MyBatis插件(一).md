官方示例

    <plugins>
        <plugin interceptor="org.mybatis.example.ExamplePlugin">
            <property name="someProperty" value="100"/>
        </plugin>
    </plugins>


1.进入XMLConfigBuilder的parseConfiguration方法

    //解析plugins标签
    pluginElement(root.evalNode("plugins"));
    ↓
    ↓
    //循环每一个plugin标签
    for (XNode child : parent.getChildren()) {
        //按照示例拿到"org.mybatis.example.ExamplePlugin"
        String interceptor = child.getStringAttribute("interceptor");
        Properties properties = child.getChildrenAsProperties();
        Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
        interceptorInstance.setProperties(properties);
        //底层就是将拦截器对象添加到一个ArrayList集合中
        configuration.addInterceptor(interceptorInstance);
    }

2.插件场景

    //初始化Executor
    executor = (Executor) interceptorChain.pluginAll(executor);

    //初始化StatementHandler
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);

    //初始化ParameterHandler
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    
    //初始化ResultSetHandler
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    
3.进入InterceptorChain的pluginAll()方法

    //遍历拦截器
    for (Interceptor interceptor : interceptors) {
      //执行拦截器的plugin方法
      target = interceptor.plugin(target);
    }
    return target;
    
---
