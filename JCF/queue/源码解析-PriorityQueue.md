#### 核心概念

```java
优先队列的作用是能保证每次取出的元素都是队列中“最小的”（小顶堆）

底层使用数组实现，父子节点关系
  leftNo = parentNo * 2 + 1
	rightNo = parentNo * 2 + 2
	parentNo = (nodeNo-1) / 2

增加元素
  元素添加到末尾
  元素如果小于父节点，变换位置

获取元素
  返回数组中第一个元素
  最后一个元素与根节点的子节点中最小的那个节点比较，如果大于，交换位置
```

---

##### 1.PriorityQueue#offer

```java
public boolean offer(E e) {
  modCount++; //通过这行代码知道PriorityQueue并不是线程安全的队列
  int i = size;

  size = i + 1; //队列中的元素个数 + 1
  if (i == 0) //如果是第一个元素直接赋值，不需要变换数组
    queue[0] = e;
  else //如果不是，则从队列的最后一个位置根据一定规则交换节点
      siftUp(i, e);
  return true;
}
↓
private void siftUp(int k, E x) {
  else //使用默认的比较器
    siftUpComparable(k, x); //1.1
}
```

##### 1.1.PriorityQueue#siftUpComparable

```java
private void siftUpComparable(int k, E x) {
  Comparable<? super E> key = (Comparable<? super E>) x;
  while (k > 0) {
    int parent = (k - 1) >>> 1; //获取父节点的位置 = (子节点位置 - 1) / 2
    Object e = queue[parent];
    if (key.compareTo((E) e) >= 0) //如果插入节点大于等于父节点，停止比较
      break;
    queue[k] = e; //如果插入节点小于父节点，插入的新节点和父节点交换位置
    k = parent;	//一层层往上循环
  }
  queue[k] = key; //数组k位置赋值为新节点
}
```

##### 2.PriorityQueue#poll

```java
public E poll() {
	int s = --size;	//元素个数-1
  modCount++;
	E result = (E) queue[0]; //获取第一个元素做为返回值，也就是要移除的元素
	E x = (E) queue[s]; //拿到最后一个元素
  queue[s] = null;
  if (s != 0)
  	siftDown(0, x); //从0开始，逐层向下与当前点的左右孩子中较小的那个交换，直到小于或等于左右孩子中的任何一个为止
	return result;
}
↓
private void siftDown(int k, E x) {
  else
    siftDownComparable(k, x); //2.1
}
```

##### 2.1.PriorityQueue#siftDownComparable

```java
private void siftDownComparable(int k, E x) {
  Comparable<? super E> key = (Comparable<? super E>)x;
  int half = size >>> 1;
  while (k < half) {
    int child = (k << 1) + 1; //获取左子节点位置 = 父节点位置 * 2 + 1
    Object c = queue[child];
    int right = child + 1;
    //这里比较左右子节点谁小，然后拿到小的那个节点，这里叫小节点
    if (right < size &&
        ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0) 
      c = queue[child = right];
    if (key.compareTo((E) c) <= 0) //如果传入的最后一个节点小于或者等于小节点就停止
      break;
    queue[k] = c; //如果大于，变换位置，把最后一个节点和小节点变换位置
    k = child;
  }
  queue[k] = key;
}
```

