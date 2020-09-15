下面所有的情况都可以自测看下MyBatis打出来的日志,通过日志都会告诉你怎么用  
我这里就是备份记忆,省的自测浪费时间了  


方法为单个参数的情况

    User selectUserById(int userId)
    ↓
    //#{任意名}
    select * from user where user_id = #{userId}
    
    User selectUserById(@Param("userId") int userId)
    ↓
    //#{注解值或者param1}
    select * from user where user_id = #{userId}
    
    User selectUserByUser(User user)
    ↓
    //#{属性名}
    select * from user where user_id = #{userId}
    
    User selectUserByUser(@Param("user") User user)
    ↓
    //#{注解值.属性名或者param1.属性名}
    select * from user where user_id = #{user.userId}
    
    User selectUserByMap(Map<String, Object> map)
    ↓
    //#{key}
    select * from user where user_id = #{userId}
    
    User selectUserByMap(@Param("map") Map<String, Object> map)
    ↓
    //#{注解值.key或者param1.key}
    select * from user where user_id = #{map.userId}
    
    User selectUserByList(List<Integer> idList)
    ↓
    //#{collection[索引]或者list[索引]}
    select * from user where user_id = #{list[0]}
    
    User selectUserByList(@Param("idList") List<Integer> idList)
    ↓
    //#{注解值[索引]或者param1[索引]}
    select * from user where user_id = #{idList[0]}

---
    
方法为多个参数的情况

    User selectUser(String name, int age)
    ↓
    //Available parameters are [0, 1, param1, param2]
    select * from user where name = #{0} and age = #{1}
    
    User selectUser(String name, Company company)
    ↓
    //Available parameters are [0, 1, param1, param2]
    select * from user where name = #{param1} and company_id = #{param2.companyId}
    
    User selectUser(List<Integer> idList, String name)
    ↓
    //Available parameters are [0, 1, param1, param2]
    select * from user where user_id = #{0[0]} and name = #{param2}
    
    User selectUser(@Param("age") int age, String name)
    ↓
    //第一个参数:#{注解值|param1}
    //第二个参数:#{1|param2}
    select * from user where age = #{age} and name = #{1}
    
    User selectUser(int age, @Param("company") Company company)
    ↓
    //第一个参数:#{0|param1}
    //第二个参数:#{注解值.属性名|param2.属性名}
    select * from user where age = #{0} and company_id = #{company.companyId}
    
    User selectUser(@Param("age") int age, @Param("company") Company company)
    ↓
    //第一个参数:#{注解值|param1}
    //第二个参数:#{注解值.属性名|param2.属性名}
    select * from user where age = #{age} and company_id = #{company.companyId}

---
