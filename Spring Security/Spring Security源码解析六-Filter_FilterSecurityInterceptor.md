#####FilterSecurityInterceptor继承关系<img src="https://user-images.githubusercontent.com/4274041/105690532-167c0a80-5f37-11eb-8c07-790068065f99.png" alt="image" style="zoom:50%;" />

1.FilterSecurityInterceptor#doFilter

```java
FilterInvocation fi = new FilterInvocation(request, response, chain);
invoke(fi);
↓
↓
InterceptorStatusToken token = super.beforeInvocation(fi);
↓
↓
this.accessDecisionManager.decide(authenticated, object, attributes);
↓
↓
//AffirmativeBased#decide
int result = voter.vote(authentication, object, configAttributes);
↓
↓
//WebExpressionVoter#vote
//这里就是SPEL表达式的东西,不展开了
return ExpressionUtils.evaluateAsBoolean(weca.getAuthorizeExpression(), ctx) ? ACCESS_GRANTED : ACCESS_DENIED;
```

2.SecurityExpressionRoot

```java
//SPEL表达式根据之前初始化的字符串authenticated会反射调用isAuthenticated方法
return !isAnonymous();
↓
↓
return trustResolver.isAnonymous(authentication);
↓
↓
//anonymousClass实现类是AnonymousAuthenticationToken
//用户登录的时候authentication实现类是UsernamePasswordAuthenticationToken,这里返回false
//上面第1步最后就返回true,代表已经验证了ACCESS_GRANTED
return anonymousClass.isAssignableFrom(authentication.getClass());
```

