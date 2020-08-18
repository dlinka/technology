循环调用add()方法的比较

| | 调用千万次 |
| ---- | ---- |
| ArrayList | 39355 |
| LinkedList | 41928 |

    没有做实验之前,我一直认为ArrayList的扩容复制开销会大一点,但是结果显示两者其实差不多
    为什么会出现这种现象呢?
    LinkedList的add()方法会大量的new对象,开销也不低

---

遍历方法的比较

| \ | fori | forEach | iterator |
| ---- | ---- | ---- | ---- |
| ArrayList | 80 | 443 | 84 |
| LinkedList | 超时 | 477 | 202 |

    通过实验结果可知
    使用iterator遍历LinkedList最好
        for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
            it.next();
        }
    使用fori遍历ArrayList最好

    为什么会有超时的现象呢?
    进入LinkedList的源码会发现有大量的for循环
        return node(index).item;
        ↓
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
