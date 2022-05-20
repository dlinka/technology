### 初始化阶段

#### 1.SpringServletContainerInitializer#onStartup

```java
public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {
  List<WebApplicationInitializer> initializers = new LinkedList<WebApplicationInitializer>();
  for (Class<?> waiClass : webAppInitializerClasses) {
    //扩展AbstractAnnotationConfigDispatcherServletInitializer类的实现了WebApplicationInitializer
    initializers.add((WebApplicationInitializer) waiClass.newInstance());
  }
  
  for (WebApplicationInitializer initializer : initializers) {
    initializer.onStartup(servletContext);
  }
}
```

#### 2.AbstractDispatcherServletInitializer#onStartup

```java
public void onStartup(ServletContext servletContext) throws ServletException {
  super.onStartup(servletContext); //3
  registerDispatcherServlet(servletContext); //4
}
```

#### 3.AbstractContextLoaderInitializer#onStartup

```java
public void onStartup(ServletContext servletContext) throws ServletException {
  registerContextLoaderListener(servletContext);
}
↓
↓
protected void registerContextLoaderListener(ServletContext servletContext) {
	//创建Spring上下文
  WebApplicationContext rootAppContext = createRootApplicationContext(); //3.1
	//创建ContextLoaderListener，容器启动会调用contextInitialized
  ContextLoaderListener listener = new ContextLoaderListener(rootAppContext);
  servletContext.addListener(listener);
}
```

#### 3.1.AbstractAnnotationConfigDispatcherServletInitializer#createRootApplicationContext

```java
protected WebApplicationContext createRootApplicationContext() {
	//调用自定义的getRootConfigClasses方法
  Class<?>[] configClasses = getRootConfigClasses();
  AnnotationConfigWebApplicationContext rootAppContext = new AnnotationConfigWebApplicationContext();
  rootAppContext.register(configClasses);
  return rootAppContext;
}
```

#### 4.AbstractDispatcherServletInitializer#registerDispatcherServlet

```java
protected void registerDispatcherServlet(ServletContext servletContext) {
	//dispatcher
  String servletName = getServletName();
  
  //创建Spring MVC上下文
  WebApplicationContext servletAppContext = createServletApplicationContext(); //4.1
  
  //创建DispatcherServlet
  FrameworkServlet dispatcherServlet = createDispatcherServlet(servletAppContext);
  
  ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);
  registration.setLoadOnStartup(1);
  //调用自定义的getServletMappings方法
  registration.addMapping(getServletMappings());
}
```

#### 4.1.AbstractAnnotationConfigDispatcherServletInitializer#createServletApplicationContext

```java
protected WebApplicationContext createServletApplicationContext() {
  AnnotationConfigWebApplicationContext servletAppContext = new AnnotationConfigWebApplicationContext();
	//调用自定义的getServletConfigClasses方法
  Class<?>[] configClasses = getServletConfigClasses();
  servletAppContext.register(configClasses);
  return servletAppContext;
}
```

### 容器启动

#### 1.ContextLoaderListener#contextInitialized

```java
public void contextInitialized(ServletContextEvent event) {
  initWebApplicationContext(event.getServletContext());
}
↓
↓
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
  if (this.context instanceof ConfigurableWebApplicationContext) {
    ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
    if (!cwac.isActive()) {
      configureAndRefreshWebApplicationContext(cwac, servletContext);
    }
  }
  //将Spring上下文设置到ServletContext上下文
  servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
}
```

#### 2.FrameworkServlet#initWebApplicationContext

```java
protected WebApplicationContext initWebApplicationContext() {
	//拿到上面创建的Spring上下文
  WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(getServletContext());
  WebApplicationContext wac = null;
  if (this.webApplicationContext != null) {
  	wac = this.webApplicationContext;
    if (wac instanceof ConfigurableWebApplicationContext) {
    	ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
				if (!cwac.isActive()) {
					if (cwac.getParent() == null) {
						//设置父子关系
						cwac.setParent(rootContext);
					}
					configureAndRefreshWebApplicationContext(cwac);
				}
    }
  }
  if (this.publishContext) {
    //将Spring MVC上下文设置到ServletContext上下文
    getServletContext().setAttribute(getServletContextAttributeName(), wac);
  }
}
```

