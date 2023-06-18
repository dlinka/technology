#### 条件语句

```python
# if elif else
a = 10
if a < 3:
		print("小于3")  
elif a < 6:
    print("小于6")
else:
    print("大于6")

# if else
a = 3
b = 5
c = a if a > b else b
```

#### 数学运算

```python
10 / 2 # 输出5.0，类型是float
5 // 2 # 地板除
2 ** 2 # 指数运算
```

#### 比较运算

```python
1 == 1 # True
1 != 1 # False
```

#### 逻辑运算

```python
# and：找到第一个False的操作数就停止，如果没有遇到False的对象，那么返回最后一个True的对象。

# or：找到第一个True的操作数就停止，如果没有遇到True的对象，那么返回最后一个False的对象。

# not
not 0 # True
not 0.0 # True
not 1 # False
not 1.1 # False
```

#### 变量

```python
a = 5

# 多个变量赋值
b, c, d = 9, 1, 1
e = f = g = 2

# 交换变量赋值
x = 5
y = 2
x, y = y, x
```

#### 优先级

```python
# 逻辑运算优先级not > and > or
True or False and False # 输出True
```

#### 类型转换

```python
int()
str()
list()
tuple()
```

#### 循环

```python
# for循环
a = [1, 2, 3]
for i in a:
    if i == 3:
        break
    elif i == 2:
        continue
    print(i)
else:
    print("正常结束")
    
# while
i = 1
while i <= 5:
    print(i)
    i += 1
    if i == 3:
        continue
    elif i == 4:
        break
else:
    print("正常结束")
```

