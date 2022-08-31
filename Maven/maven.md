```shell
#不执行测试用例,但编译测试用例生成相应的class文件至target/test-classes下
mvn clean install -DskipTests

#不但不执行测试用例,也跳过测试代码的编译
mvn clean install -Dmaven.test.skip=true
```

---

### 插件

#### dependency

```shell
#查找所有groupId前缀是org.apache.
mvn dependency:tree -Dincludes=org.apache.*

#查找artifactId为log4j
mvn dependency:tree -Dincludes=:log4j
  
#查找所有SNAPSHOT
mvn dependency:tree -Dincludes="::*-SNAPSHOT"
```



---

[Maven最佳实践:版本管理](https://www.iteye.com/blog/juvenshun-376422)
