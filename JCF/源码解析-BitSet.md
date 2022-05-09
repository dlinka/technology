##### 面试题

```
如何对10亿个电话号码去重复处理？

号码都是1开头
号码分段135、136等
```

---

#### 源码解析

##### 1.BitSet构造方法

```java
public BitSet() {
  //BITS_PER_WORD等于64(1 << 6)
  initWords(BITS_PER_WORD);
  sizeIsSticky = false;
}
↓
↓
private void initWords(int nbits) {
  words = new long[wordIndex(nbits-1) + 1];
}
↓
↓
//计算数组下标
private static int wordIndex(int bitIndex) {
  //63 >> 6 = 0
  //129 >> 6 = 2
  return bitIndex >> ADDRESS_BITS_PER_WORD;
}
```

##### 2.set

```java
public void set(int bitIndex) {
  //计算下标
  int wordIndex = wordIndex(bitIndex);
  
  expandTo(wordIndex);
  
  //1L << bitIndex相当于bitIndex % 64
  words[wordIndex] |= (1L << bitIndex);
}
↓
↓
private void expandTo(int wordIndex) {
  int wordsRequired = wordIndex + 1;
  if (wordsInUse < wordsRequired) {
		//扩容
    ensureCapacity(wordsRequired);
    //wordsInUse表示使用的了几个words，等于words数组下标最大值 + 1
    wordsInUse = wordsRequired;
  }
}
↓
↓
private void ensureCapacity(int wordsRequired) {
  if (words.length < wordsRequired) {
    int request = Math.max(2 * words.length, wordsRequired);
    words = Arrays.copyOf(words, request);
    sizeIsSticky = false;
  }
}
```

##### 3.length

```java
public int length() {
  return BITS_PER_WORD * (wordsInUse - 1) +
    (BITS_PER_WORD - Long.numberOfLeadingZeros(words[wordsInUse - 1]));
}
```

---