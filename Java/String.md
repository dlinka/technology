```java
使用String.valueOf替代""+value
  
使用split(String regex)方法,部分关键字需要转译
  String[] sa1 = "a.ab.abc".split("."); //[]
	String[] sa2 = "a.ab.abc".split("\\."); //[a, ab, abc]

	String[] sa1 = "a|ab|abc".split("|"); //[a, |, a, b, |, a, b, c]
	String[] ss2 = "a|ab|abc".split("\\|"); //[a, ab, abc]
```

