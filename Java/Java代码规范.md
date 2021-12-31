##### 少用!

```java
//错误示范
if (length < 10 || !flag)
//正确示范
if(length >= 10 && flag)
```

##### 返回空集合代替null

```java
Collections.emptyList()
Collections.emptyMap()
```

