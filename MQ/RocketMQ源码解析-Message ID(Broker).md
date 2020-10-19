SendResult [sendStatus=SEND_OK, msgId=0A00C334F59A18B4AAC25F363E9D0000, offsetMsgId=0A00C33400002A9F0000000000084476, messageQueue=MessageQueue [topic=TopicTest, brokerName=cyg.local, queueId=2], queueOffset=653]

---

offsetMsgId的生成规则

    SendResult sendResult = new SendResult(sendStatus, uniqMsgId, responseHeader.getMsgId(), messageQueue, responseHeader.getQueueOffset());
    ↓
    ↓
    public SendResult(SendStatus sendStatus, String msgId, String offsetMsgId, MessageQueue messageQueue,
        long queueOffset) {
        this.sendStatus = sendStatus;
        this.msgId = msgId;
        this.offsetMsgId = offsetMsgId;
        this.messageQueue = messageQueue;
        this.queueOffset = queueOffset;
    }

1.进入MessageDecoder的decode方法

    ByteBuffer byteBufferMsgId = ByteBuffer.allocate(msgIDLength);
    String msgId = createMessageId(byteBufferMsgId, msgExt.getStoreHostBytes(), msgExt.getCommitLogOffset());
    ↓
    ↓
    //msgId包含地址和偏移量
    input.put(addr);
    input.putLong(offset);
    return UtilAll.bytes2string(input.array());

---
