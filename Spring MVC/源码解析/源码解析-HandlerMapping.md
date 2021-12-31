#### 1.DispatcherServlet的initStrategies方法

```java
this.initHandlerMappings(context);
```

#### 2.DispatcherServlet的initHandlerMappings方法

```java
Map hm = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
if(!hm.isEmpty()) {
  //SimpleUrlHandlerMapping,BeanNameUrlHandlerMapping,RequestMappingHandlerMapping
  this.handlerMappings = new ArrayList(hm.values());
  //排序(RequestMappingHandlerMapping变为第一个元素)
  AnnotationAwareOrderComparator.sort(this.handlerMappings);
}
```

#### 3.请求到来之后,执行DispatcherServlet的doDispatch方法

```java
//HandlerExecutionChain
mappedHandler = this.getHandler(processedRequest);
```

#### 4.DispatcherServlet的getHandler方法

```java
//循环上面三个HandlerMapping
HandlerMapping hm = (HandlerMapping)var2.next();
//HandlerExecutionChain
handler = hm.getHandler(request);
```

#### 5.以RequestMappingHandlerMapping为例,进入AbstractHandlerMapping的getHandler方法

```java
//调用子类AbstractHandlerMethodMapping的getHandlerInternal方法返回HandlerMethod对象
Object handler = this.getHandlerInternal(request);
//调用AbstractHandlerMapping的getHandlerExecutionChain方法返回HandlerExecutionChain对象
HandlerExecutionChain executionChain1 = this.getHandlerExecutionChain(handler, request);
```

