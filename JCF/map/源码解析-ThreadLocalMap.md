为了有助于理解源码，参考下面数组，3、5、9位置此刻已经被GC，key为null
假设hash函数是对10取余，执行set(25, value)

| 下标  | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| key   |      |      | 12   | null | 4    | null | 15   | 25   |      | null |
| value |      |      | v12  | v13  | v4   | v5   | v15  | v25  |      | v19  |

---

#### 1.set

```java
private void set(ThreadLocal<?> key, Object value) {
	Entry[] tab = table;
  int len = tab.length;
  int i = key.threadLocalHashCode & (len-1); //i=5
 
 	for (Entry e = tab[i];
       e != null;
       e = tab[i = nextIndex(i, len)]) { 
  	ThreadLocal<?> k = e.get();

    if (k == key) { //如果相等，赋值value，直接返回
      e.value = value;
      return;
    }
    
    if (k == null) { //如果这个位置被GC了
      replaceStaleEntry(key, value, i); //2
      return;
    }
  }
  
  tab[i] = new Entry(key, value); //如果这个位置为null，初始化Entry
}

```

#### 2.replaceStaleEntry

```java
private void replaceStaleEntry(ThreadLocal<?> key, Object value, int staleSlot) {
	Entry[] tab = table;
  int len = tab.length;
  Entry e;
  
  int slotToExpunge = staleSlot;
  for (int i = prevIndex(staleSlot, len); //从5的位置往前直到数组没有被占用的位置为止，正常到1的位置结束循环
       (e = tab[i]) != null;
       i = prevIndex(i, len))
    if (e.get() == null)
      slotToExpunge = i; //slotToExpunge=3
  
  
  for (int i = nextIndex(staleSlot, len); //从5的位置往后直到数组位置没有被占用为止，正常到8的位置结束循环
       (e = tab[i]) != null;
       i = nextIndex(i, len)) {
    ThreadLocal<?> k = e.get();
  	if (k == key) { //i=6，k=15
      e.value = value; //位置6的value变成新的value

      //位置5和位置6互换
      tab[i] = tab[staleSlot];
      tab[staleSlot] = e;

      cleanSomeSlots(expungeStaleEntry(slotToExpunge), len); //expungeStaleEntry(slotToExpunge)返回8
      return;
    }
  }
  
  //没有找到，会直接使用位置5
  tab[staleSlot].value = null;
  tab[staleSlot] = new Entry(key, value);
}
```

#### 3.expungeStaleEntry

```java
private int expungeStaleEntry(int staleSlot) {
	Entry[] tab = table;
	int len = tab.length;

  //位置3清除
	tab[staleSlot].value = null;
	tab[staleSlot] = null;
	size--;
  
  Entry e;
  int i;
  for (i = nextIndex(staleSlot, len); //从位置4往后直到数组没有被占用的位置为止，正常到位置8结束循环
       (e = tab[i]) != null;
       i = nextIndex(i, len)) {
  	ThreadLocal<?> k = e.get();
    if (k == null) { //位置6清除
      e.value = null;
      tab[i] = null;
      size--;
    } else {
      int h = k.threadLocalHashCode & (len - 1); //h=5
      if (h != i) { //位置7的key哈希和h不相等
        tab[i] = null; //位置7清除
        
        while (tab[h] != null) //从位置5查找第一个为null的位置
          h = nextIndex(h, len);
        tab[h] = e; //位置7移动到位置6
      }
    }
  }
  return i;
}
```

#### 4.cleanSomeSlots

```java
private boolean cleanSomeSlots(int i, int n) {
	boolean removed = false;
	Entry[] tab = table;
	int len = tab.length;
  do {
    i = nextIndex(i, len); //i=9
    Entry e = tab[i];
    if (e != null && e.get() == null) { //这步会清空位置9
      n = len; //这里n重新赋值会重新循环5次
      removed = true;
      i = expungeStaleEntry(i); //参考上面3
    }
  } while ( (n >>>= 1) != 0); //默认n等于2的4次方，也就是位移5次才会等于0，也就是这里循环5次
	return removed;
}
```

