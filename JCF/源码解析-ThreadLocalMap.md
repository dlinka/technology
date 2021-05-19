| 下标  | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| key   |      |      | 12   | null | 4    | null | 15   | 25   |      | null |
| value |      |      | v12  | v13  | v4   | v5   | v15  | v25  |      | v19  |

**根据上面数组情况分析源码,假设hash是对10取余,3、5、9位置此刻被垃圾回收**

**set(15,v1515)**

---

**1.set(ThreadLocal<?> key, Object value)**

```java
Entry[] tab = table;
int len = tab.length;

//计算数组下标
//i = 5
int i = key.threadLocalHashCode & (len-1);

//这里我改造了一下代码,方便阅读
Entry e = tab[i];
while(e != null) {
  ThreadLocal<?> k = e.get();
  
  //如果相等,直接赋值返回就行了
  if (k == key) {
  	e.value = value;
    return;
  }
  
  //如果这个位置被垃圾回收了
  //i = 5
  if (k == null) {
  	replaceStaleEntry(key, value, i); //1.1
    return;
  }
  
  //下一个位置
  i = nextIndex(i, len);
  e = tab[i];
}
```

**1.1replaceStaleEntry(ThreadLocal<?> key, Object value, int staleSlot)**

```java
Entry[] tab = table;
int len = tab.length;
Entry e;

int slotToExpunge = staleSlot;
//从5的位置往前直到数组位置没有被占用为止
for(int i = prevIndex(staleSlot, len); (e = tab[i]) != null; i = prevIndex(i, len)) {
  if (e.get() == null)
    //slotToExpunge = 3
    slotToExpunge = i;
}

//从5的位置往后直到数组位置没有被占用为止
for (int i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
	ThreadLocal<?> k = e.get();
  //找出和当前匹配的key
  if (k == key) {
    //6位置的值变成新值
  	e.value = value;
    //5和6位置上元素互换
    tab[i] = tab[staleSlot];
    tab[staleSlot] = e;
    
    //从上面标记位置开始清除
    //slotToExpunge = 3
		int j = expungeStaleEntry(slotToExpunge); //1.1.1
    cleanSomeSlots(j, len); //1.1.2
    return;
  }
}
```

**1.1.1expungeStaleEntry(int staleSlot)**

```java
Entry[] tab = table;
int len = tab.length;

//3位置清除
tab[staleSlot].value = null;
tab[staleSlot] = null;
size--;

Entry e;
int i;
//从4位置开始
for (i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
	ThreadLocal<?> k = e.get();
  if (k == null) {
    //6位置被清除
    e.value = null;
    tab[i] = null;
    size--;
  } else {
    int h = k.threadLocalHashCode & (len - 1);
    if (h != i) {
      //7位置才会到这里
      //h = 5, i = 7
    	tab[i] = null;
      while (tab[h] != null)
      	h = nextIndex(h, len);
      //7位置元素赋值到6位置
      //h = 6
      tab[h] = e;
    }
	}
}
//返回7
return i;
```

**1.1.2cleanSomeSlots(int i, int n)**

**这步是为了清除9**

```java
boolean removed = false;
Entry[] tab = table;
int len = tab.length;
do {
    //从8位置开始
    i = nextIndex(i, len);
    Entry e = tab[i];
    if (e != null && e.get() == null) {
        //这里会重新开始循环
        n = len;
        removed = true;
        //参考上面1.1.1
        i = expungeStaleEntry(i);
    }
} while ( (n >>>= 1) != 0); //默认n是16,这里会循环5次
return removed;
```

