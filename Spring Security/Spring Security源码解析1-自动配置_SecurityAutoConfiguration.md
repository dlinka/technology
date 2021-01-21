1.spring.factories

```java
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
```

2.SecurityAutoConfiguration

```java
//仅在DefaultAuthenticationEventPublisher存在于classpath上时才进行配置
@ConditionalOnClass(DefaultAuthenticationEventPublisher.class)
@EnableConfigurationProperties(SecurityProperties.class)
@Import({ 
	SpringBootWebSecurityConfiguration.class, //3
	WebSecurityEnablerConfiguration.class, //4
	SecurityDataConfiguration.class
})
public class SecurityAutoConfiguration {
}
```

3.SpringBootWebSecurityConfiguration

```java
@ConditionalOnClass(WebSecurityConfigurerAdapter.class)
//如果没有自定义配置类,这个类就会为其提供一个默认的配置类
//所以自定义的配置类需要继承WebSecurityConfigurerAdapter
@ConditionalOnMissingBean(WebSecurityConfigurerAdapter.class)
public class SpringBootWebSecurityConfiguration {
	@Configuration(proxyBeanMethods = false)
	@Order(SecurityProperties.BASIC_AUTH_ORDER)
	static class DefaultConfigurerAdapter extends WebSecurityConfigurerAdapter { 
  }
}
```

4.WebSecurityEnablerConfiguration

```java
//仅存在类型为WebSecurityConfigurerAdapter的Bean才生效
@ConditionalOnBean(WebSecurityConfigurerAdapter.class)
//仅在名为springSecurityFilterChain的Bean不存在时才生效
@ConditionalOnMissingBean(name = BeanIds.SPRING_SECURITY_FILTER_CHAIN)
@EnableWebSecurity
public class WebSecurityEnablerConfiguration {
}
↓
↓
@Import({
  WebSecurityConfiguration.class, //5
	SpringWebMvcImportSelector.class,
	OAuth2ImportSelector.class
})
@EnableGlobalAuthentication //6
public @interface EnableWebSecurity {
}
```

5.WebSecurityConfiguration

```java
//注入名为springSecurityFilterChain的Bean
@Bean(name = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME)
public Filter springSecurityFilterChain() throws Exception {
}
```

6.EnableGlobalAuthentication

```java
@Import(AuthenticationConfiguration.class)
public @interface EnableGlobalAuthentication {
}
```

