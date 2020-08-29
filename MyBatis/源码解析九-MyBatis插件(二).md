官网示例

    @Intercepts({
            @Signature(
                type= Executor.class,
                method = "update",
                args = {MappedStatement.class, Object.class})
    })
    public class ExamplePlugin implements Interceptor {
        @Override
        public Object intercept(Invocation invocation) throws Throwable {
            // implement pre-processing if needed
            Object returnObject = invocation.proceed();
            // implement post-processing if needed
            return returnObject;
        }
        
        @Override
        public Object plugin(Object target) {
            return Plugin.warp(target, this);
        }
    }
    
1.进入Plugin的warp方法

    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor); //2
    Class<?> type = target.getClass();
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    if (interfaces.length > 0) {
      return Proxy.newProxyInstance(
          type.getClassLoader(),
          interfaces,
          new Plugin(target, interceptor, signatureMap)); //3
    }

2.进入getSignatureMap方法
    
    //获取当前拦截器的@Intercepts注解
    Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
    //获取所有Signature
    Signature[] sigs = interceptsAnnotation.value();
    Map<Class<?>, Set<Method>> signatureMap = new HashMap<Class<?>, Set<Method>>();
    for (Signature sig : sigs) {
        Set<Method> methods = signatureMap.get(sig.type());
        if (methods == null) {
            methods = new HashSet<Method>();
            //key为Executor.class
            signatureMap.put(sig.type(), methods);
        }
        Method method = sig.type().getMethod(sig.method(), sig.args());
        //把Executor的update方法放到methods集合中
        methods.add(method);
    }
    return signatureMap;

3.进入invoke方法

    Set<Method> methods = signatureMap.get(method.getDeclaringClass());
    if (methods != null && methods.contains(method)) {
        //如果注解的中方法包含当前方法,进入拦截器的intercept方法
        return interceptor.intercept(new Invocation(target, method, args));
    }

---
