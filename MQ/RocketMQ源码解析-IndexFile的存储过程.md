1.进入DefaultMessageStore的doReput方法

    DispatchRequest dispatchRequest = DefaultMessageStore.this.commitLog.checkMessageAndReturnSize(result.getByteBuffer(), false, false); //2
    ...
    DefaultMessageStore.this.doDispatch(dispatchRequest);
    ↓
    ↓
    dispatcher.dispatch(req);
    ↓
    ↓
    //进入CommitLogDispatcherBuildIndex的dispatch方法
    DefaultMessageStore.this.indexService.buildIndex(request);
    ↓
    ↓
    //进入IndexService的buildIndex方法
    //UNIQ_KEY写入IndexFile
    if (req.getUniqKey() != null) {
        //生成因子加入Topic
        indexFile = putKey(indexFile, msg, buildKey(topic, req.getUniqKey()));
    }
    //KEYS写入IndexFile
    if (keys != null && keys.length() > 0) {
        String[] keyset = keys.split(MessageConst.KEY_SEPARATOR);
        for (int i = 0; i < keyset.length; i++) {
            String key = keyset[i];
            indexFile = putKey(indexFile, msg, buildKey(topic, key));
        }
    }
    ↓
    ↓
    //进入IndexService的putKey方法
    boolean ok = indexFile.putKey(idxKey, msg.getCommitLogOffset(), msg.getStoreTimestamp()); //3
    
2.1进入CommitLog的checkMessageAndReturnSize方法

    //UNIQ_KEY
    uniqKey = propertiesMap.get(MessageConst.PROPERTY_UNIQ_CLIENT_MESSAGE_ID_KEYIDX);
    
3.进入IndexFile的putKey方法

    //hashcode
    int keyHash = indexKeyHashMethod(key);
    //对500万取余
    int slotPos = keyHash % this.hashSlotNum;
    //40 + 4 * slotPos
    int absSlotPos = IndexHeader.INDEX_HEADER_SIZE + slotPos * hashSlotSize;
    ...
    
    //获取slot值
    int slotValue = this.mappedByteBuffer.getInt(absSlotPos);
    //slot值不存在就为0
    if (slotValue <= invalidIndex || slotValue > this.indexHeader.getIndexCount()) {
        slotValue = invalidIndex;
    }
    
    long timeDiff = storeTimestamp - this.indexHeader.getBeginTimestamp();
    timeDiff = timeDiff / 1000;
    
    //index的位置
    //源码可以看出来第一个位置空了出来
    int absIndexPos = IndexHeader.INDEX_HEADER_SIZE + this.hashSlotNum * hashSlotSize + this.indexHeader.getIndexCount() * indexSize;
    //存放hashcode
    this.mappedByteBuffer.putInt(absIndexPos, keyHash);
    //存放偏移量
    this.mappedByteBuffer.putLong(absIndexPos + 4, phyOffset);
    //存放时间差
    this.mappedByteBuffer.putInt(absIndexPos + 4 + 8, (int) timeDiff);
    //存放前一个index的位置,第一个存放0
    this.mappedByteBuffer.putInt(absIndexPos + 4 + 8 + 4, slotValue);
    
    //solt存放当前index的数量
    this.mappedByteBuffer.putInt(absSlotPos, this.indexHeader.getIndexCount());
    
    //如果是第一个写入Header
    if (this.indexHeader.getIndexCount() <= 1) {
        this.indexHeader.setBeginPhyOffset(phyOffset);
        this.indexHeader.setBeginTimestamp(storeTimestamp)
    }

    //如果是未使用的solt,增加solt个数
    if (invalidIndex == slotValue) {
        this.indexHeader.incHashSlotCount();
    }
    
    //增加Header中index的个数
    this.indexHeader.incIndexCount();
    this.indexHeader.setEndPhyOffset(phyOffset);
    this.indexHeader.setEndTimestamp(storeTimestamp);

---
    
