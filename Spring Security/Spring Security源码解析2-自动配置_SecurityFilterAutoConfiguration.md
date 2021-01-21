1.spring.factories

```java
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration
```

2.SecurityFilterAutoConfiguration

```java
@EnableConfigurationProperties(SecurityProperties.class)
//指定在SecurityAutoConfiguration配置之后
@AutoConfigureAfter(SecurityAutoConfiguration.class)
public class SecurityFilterAutoConfiguration {
  //这个Bean是用来注册Filter到ServletContext中
	@Bean
	@ConditionalOnBean(name = DEFAULT_FILTER_NAME)
	public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(SecurityProperties securityProperties) {
		DelegatingFilterProxyRegistrationBean registration = new DelegatingFilterProxyRegistrationBean(DEFAULT_FILTER_NAME);
    ...
		return registration;
	}
}
```

---

DelegatingFilterProxyRegistrationBean的继承关系

![DelegatingFilterProxyRegistrationBean继承关系](https://user-images.githubusercontent.com/4274041/80004215-a26a8d00-84f4-11ea-9bf6-68df919a770b.png)

1.RegistrationBean#onStartup

由于DelegatingFilterProxyRegistrationgBean父类RegistrationBean实现了ServletContextInitializer接口

容器启动的时候会调用ServletContextInitializer实现类的onStartup方法

```java
register(description, servletContext);
```

2.DynamicRegistrationBean#register

```java
D registration = addRegistration(description, servletContext);
```

3.AbstractFilterRegistrationBean#addRegistration

```java
Filter filter = getFilter();
//这里就会把Filter注入到ServletContext中
return servletContext.addFilter(getOrDeduceName(filter), filter);
```

4.DelegatingFilterProxyRegistrationBean#getFilter

```java
//DelegatingFilterProxy是一个Filter
//DelegatingFilterProxy本质是一个代理类,代理名为springSecurityFilterChain的Bean
return new DelegatingFilterProxy(this.targetBeanName, getWebApplicationContext()) {
	@Override
	protected void initFilterBean() throws ServletException {
		// Don't initialize filter bean on init()
	}
};
```

