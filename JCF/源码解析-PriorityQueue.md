1.offer

```java
//通过这行代码知道PriorityQueue并不是线程安全的队列
modCount++;
//size表示队列中的元素个数
int i = size;
if (i >= queue.length)
    //扩容
    grow(i + 1);
size = i + 1;
//如果是第一个元素直接赋值,不需要变换数组
if (i == 0)
    queue[0] = e;
else
    //i等于队列长度,表示从后开始交换节点
    siftUp(i, e);
return true;
↓
↓
siftUpComparable(k, x);
↓
↓
Comparable<? super E> key = (Comparable<? super E>) x;
while (k > 0) {
    //获取父节点的位置
    //父节点位置 = (子节点位置 - 1) / 2
    int parent = (k - 1) >>> 1;
    Object e = queue[parent];
    //如果插入节点大于父节点,停止比较
    if (key.compareTo((E) e) >= 0)
        break;
    //如果插入节点小于父节点,插入节点和父节点交换位置(其实没有实际交换)
    queue[k] = e;
    //插入节点就变成父节点位置开启下一次循环
    k = parent;
}
//数组k位置赋值为key
queue[k] = key;
```

2.poll

```java
//元素个数-1
int s = --size;
modCount++;
//获取第一个元素做为返回值
E result = (E) queue[0];
//最后一个元素
E x = (E) queue[s];
//最后一个位置复制为null
queue[s] = null;
if (s != 0)
    //这里可以先把最后一个节点移动到了根节点,方便理解接下来的代码
    siftDown(0, x);
return result;
↓
↓
siftDownComparable(k, x);
↓
↓
//无符号右移1位
int half = size >>> 1;
while (k < half) {
    //获取左子节点位置
    //左子节点位置 = 父节点位置 * 2 + 1
    int child = (k << 1) + 1;
    Object c = queue[child];
    int right = child + 1;
    //这里比较左右子节点谁小,然后拿到小的那个节点,这里叫小节点
    if (right < size &&
        ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
        c = queue[child = right];
    //如果最后一个节点小于或者等于小节点就停止
    if (key.compareTo((E) c) <= 0)
        break;
    //如果大于
    //变换位置,把最后一个节点和小节点变换位置
    queue[k] = c;
    k = child;
}
queue[k] = key;
```

3.remove(Object o)

```java
//循环遍历,找到第一个相等的节点,时间复杂度O(n)
int i = indexOf(o);
if (i == -1)
    return false;
else {
    removeAt(i);
    return true;
}
↓
↓
int s = --size;
//如果是最后一个元素直接赋值为null即可
if (s == i)
    queue[i] = null;
else {
    //移动最后一个元素
    E moved = (E) queue[s];
    queue[s] = null;
    //按照poll的方法移动
  	siftDown(i, moved);
}
return null; 
```

