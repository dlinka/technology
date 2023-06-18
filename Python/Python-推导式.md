#### 推导式

```python
# 得到10以内的偶数列表
a = [i for i in range(10) if i % 2 == 0]

# 将两个列表合并成一个字典
a = ['apple', 'orange']
b = [1, 2]
c = {a[i]: b[i] for i in range(2)}

# 提取大于1000的品牌字典
a = {'apple': 1000, 'oppo': 2000}
b = {k: v for k, v in a.items() if v > 1000}

# 根据列表生成集合
a = [2, 2, 3]
b = {i ** 3 for i in a}
```

