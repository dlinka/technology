```java
Connection conn = null;
Statement stmt = null;
conn = ...;
stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("XXX");
while(rs.next()){
  str = rs.getString("XXX");
  ResultSet rs1 = stmt.executeQuery("XXX");
}
↓
↓
一个Statement对象同时只能有一个结果集在活动,就是说即使没有调用ResultSet的close()方法,只要打开第二个结果集就隐含着对上一个结果集的关闭
所以
如果想同时对多个结果集操作,就要创建多个Statement对象,如果不需要同时操作,那么可以在一个Statement对象上须序操作多个结果集
  
当第二次调用stmt.executeQuery("XXX"),赋给rs1时,这时隐含的操作是已经关闭了rs,循环失败
  
所以如果要同时操作多个结果集一定要让它他绑定到不同的Statement对象上,好在一个connection对象可以创建任意多个Statement对象,而不需要你重新获取连接
```

