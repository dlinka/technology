## 宽松绑定

下面的变体都会绑定到hostName上

```
mail.hostName
mail.hostname
mail.host_name
mail.host-name
mail.HOST_NAME
```

## 属性

ignoreInvalidFields

```
属性配置错误的值时，而又不希望启动失败，可以设置ignoreInvalidFields属性为true (默认为false)
如为布尔值的属性提供的值为'foo'
```

## 集合属性

List和Set

```properties
mail.server[0]=server1
mail.server[1]=server1
```

```yaml
mail:
	server:
		- server1
		- server2
```

## 

