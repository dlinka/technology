#### str

```python
# 多行字符串
str = """hello
world"""

print(str[0])
print(str[开始位置:结束位置:步长]) #切片
print(str.find("a")) # 找不到返回-1
print(str.index("a")) # 找不到报错
print(str.count("a"))
print(str.replace("a", "1"))
print(str.split("l"))
print(",",join("hello", "world"))
```

#### list

```python
item = ["a", "b", "c", "d"]
print(item[1])
print(item[0:1])
print(item.index("e"))  # 报错
print(item.count("a"))
print(len(item))

item.append("e")  # 只能增加一个
item.extend(["f", "g"])  # 增加多个
item.insert(0, "z")

del item
del item[1]
item.pop()
item.remove("a") #移除第一个匹配项
item.clear()

item[0] = "aa"
print(item)
item.reverse()
item1.sort(reverse=True)

for i in item:
    print(i)
```

#### tuple

```python
item = ("a", "b", "c", "d")
single_item = (1,)

# 元组内数据不可以改变，如果想修改，请使用列表。如果必须修改，可以参考下面重新赋值案例。
item1 = ("a", "b", "c", "d")
item2 = item1[0:1] + ("bb",) + item1[3:4]
print(item2)
```

#### dict

```python
_dict1 = {'a': 1, 'b': 2}

_dict2 = {} # 空字典
_dict3 = dict() # 空字典

print(type(_dict1)) # <class 'dict'>

_dict1['c'] = 3

print(_dict1)
print(_dict1['a'])
print(_dict1.get('d', 11))

for k in _dict1.keys():
    print(k)
for v in _dict1.values():
    print(v)
for i in _dict1.items():
    print(i)
for k, v in _dict1.items():
    print(f'{k}, {v}')

del _dict1['a']
print(_dict1)
```

#### set

```python
_set1 = {'cc', 'bb', 'aa', 'aa', 'zz', 'yyy'}
print(_set1)  # 顺序随机，自动去除重复数据
_set1.add('abc')
_set1.update('123')  # 添加字符串
_set1.update(['4', '5', '6'])  # 添加元组
print(_set1)
_set1.discard('1')
print(_set1)
print('abc' in _set1)
print('xyz' not in _set1)

_str = 'asdf'
print(set(_str))  # 将字符序列转换成集合

_set2 = set()

```

