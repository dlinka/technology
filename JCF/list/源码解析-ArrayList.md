```java
初始化List的时候指定其大小,这样可以减少List的扩容次数
List的contains方法时间复杂度为O(n),可以将List转成Set,将时间复杂度变为O(1)
```

---

#### 1.ArrayList的构造方法

```java
public ArrayList() {
    //DEFAULTCAPACITY_EMPTY_ELEMENTDATA是一个Object类型的空数组
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

#### 2.add

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);
    elementData[size++] = e;
    return true;
}
↓
↓
//ensureCapacityInternal方法
private void ensureCapacityInternal(int minCapacity) {
		ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

#### 3.calculateCapacity

```java
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    //第一次返回10
  	if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
```

#### 4.ensureExplicitCapacity

```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

#### 5.grow

```java
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
  	//不考虑这种情况
    if (newCapacity - MAX_ARRAY_SIZE > 0)
      	...
    //复制
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

#### 6.remove

```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
↓
↓
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
      	//复制指定源数组src到目标数组dest,复制从src的srcPos索引开始,复制的个数是length,复制到dest的索引从destPos开始
      	//arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null;
}
```



---

### Itr

#### 属性

```java
int cursor; //游标,初始化是0
int lastRet = -1; //上一个位置,初始化等于-1
int expectedModCount = modCount; //初始化等于modCount
```

#### 1.hasNext

```java
public boolean hasNext() {
  	//游标初始化是0
    return cursor != size;
}
```

#### 2.next

```java
public E next() {
    checkForComodification();
  	//游标初始化是0
    int i = cursor;
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    //游标 + 1
  	cursor = i + 1;
    //lastRet记录上一个位置
    return (E) elementData[lastRet = i];
}
```

#### 3.checkForComodification

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

#### 4.remove

```java
public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();
    try {
        ArrayList.this.remove(lastRet);
        cursor = lastRet;
        lastRet = -1;
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```



---

### ConcurrentModificationException异常

```java
List<Integer> list = new ArrayList<>();
list.add(new Integer(1));
list.add(new Integer(2));
Iterator<Integer> iterator = list.iterator();
while (iterator.hasNext()){
    Integer next = iterator.next();
    if(next.equals(2)){
        list.remove(next);
    }
}
↓
↓
Itr初始化后expectedModCount属性值等于modCount属性值
while循环内调用ArrayList的remove方法中会修改modCount属性值
再次调用next方法执行checkForComodification方法就会抛出异常
↓
↓
单线程可以在循环中使用迭代器的remove方法(iterator.remove()),而不是使用ArrayList的remove方法
因为迭代器的remove方法中会执行expectedModCount = modCount,重新赋值expectedModCount
```

### 并发ConcurrentModificationException异常

```java
ArrayList<Integer> list = new ArrayList<Integer>();
list.add(1);
list.add(2);
list.add(3);
list.add(4);
list.add(5);

Thread thread1 = new Thread(() -> {
    Iterator<Integer> iterator = list.iterator();
    while (iterator.hasNext()) {
        Facility.print("read - {}", iterator.next());
        Facility.sleep(1);
    }
});

Thread thread2 = new Thread(() -> {
    Iterator<Integer> iterator = list.iterator();
    while (iterator.hasNext()) {
        Integer integer = iterator.next();
        if (integer == 3)
            Facility.print("remove - {}", integer);
            iterator.remove();
    }
});

thread1.start();
thread2.start();
↓
↓
thread2中iterator和thread1中的iterator是两个不同的对象
thread2中调用remove方法,会改变modCount
thread1中调用next方法,判断的是thread1中expectedModCount和整个ArrayList的modCount的值,所以抛出异常
```

```java
ArrayList<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);
list.add(4);
list.add(5);

Thread thread1 = new Thread(() -> {
		//第一种read抛出ConcurrentModificationException异常
  	for (Integer i : list) {
        Facility.print("read - {}",i);
        Facility.sleep(1);
    }
  	
  	//第二种read也抛出ConcurrentModificationException异常
  	Iterator<Integer> iterator = list.iterator();
		while (iterator.hasNext()) {
    		Facility.print("read - {}", iterator.next());
    		Facility.sleep(1);
		}
});

Thread thread2 = new Thread(() -> {
    int i = 6;
    list.add(i);
  	//write
    Facility.print("write - {}", i);
});

thread1.start();
thread2.start();
↓
↓
同上一个例子抛出异常是一个道理
↓
↓
使用CopyOnWriteArrayList并发集合或者使用锁来控制
```

---



