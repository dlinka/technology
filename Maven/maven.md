### 插件

#### dependency

```java
//查找所有groupId前缀是org.apache.
mvn dependency:tree -Dincludes=org.apache.*

//查找artifactId为log4j
mvn dependency:tree -Dincludes=:log4j
  
//查找所有SNAPSHOT
mvn dependency:tree -Dincludes=":::*-SNAPSHOT"
```



---

[Maven最佳实践:Maven仓库](https://www.iteye.com/blog/juvenshun-359256)  
[Maven最佳实践:版本管理](https://www.iteye.com/blog/juvenshun-376422)
