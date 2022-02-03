```java
Connection conn = null;
Statement stmt = null;
conn = ...;
stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("XXX");
while(rs.next()){
  str = rs.getString("XXX");
  //当第二次调用stmt.executeQuery("XXX"),赋给rs1时,这时隐含的操作是已经关闭了rs,循环失败
  ResultSet rs1 = stmt.executeQuery("XXX");
}
↓
↓
一个Statement同时只能有一个ResultSet在活动
同一个Statement生成第二个ResultSet就会隐式对上一个ResultSet的关闭

所以如果想同时对多个ResultSet操作,就要创建多个Statement对象
如果不需要同时操作,那么可以在一个Statement对象上顺序操作多个ResultSet

所以如果要同时操作多个ResultSet一定要让它他绑定到不同的Statement对象上,好在一个connection对象可以创建任意多个Statement对象,而不需要你重新获取连接
```

