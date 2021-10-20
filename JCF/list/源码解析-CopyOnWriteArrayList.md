### 特点

```java
1.线程安全
2.写的时候加锁
3.迭代器中循环的是"老数组",所以读取不到新加入的元素
```



---

#### 1.CopyOnWriteArrayList的构造函数

```java
public CopyOnWriteArrayList() {
    setArray(new Object[0]);
}
↓
↓
final void setArray(Object[] a) {
    array = a;
}
```

#### 2.add

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
  	//并发控制写
    lock.lock();
    try {
      	//返回当前数组
        Object[] elements = getArray();
        int len = elements.length;
      	//创建新的数组
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
      	//新的数组替代老的数组
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```



---

### COWIterator

#### COWIterator的构造函数

```java
private COWIterator(Object[] elements, int initialCursor) {
    cursor = initialCursor;
  	//snapshot指向当前数组
  	snapshot = elements;
}
```

#### next

```java
public E next() {
    if (! hasNext())
        throw new NoSuchElementException();
  	//调用add方法,内部创建新的数组对这里不影响
    return (E) snapshot[cursor++];
}
```



