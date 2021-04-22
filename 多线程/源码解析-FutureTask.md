cancel

```java
任务未开始
  无论mayInterruptIfRunning值是什么,返回true
任务进行中
  mayInterruptIfRunning = true,返回true
  mayInterruptIfRunning = false,返回false
任务已完成
  无论mayInterruptIfRunning值是什么,返回false
```



