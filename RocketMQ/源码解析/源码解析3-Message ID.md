**Producer发送消息成功后的返回值**

```
SendResult [sendStatus=SEND_OK, msgId=0A00C07AC3B718B4AAC230B977520000, offsetMsgId=0A00C07A00002A9F0000000000000000, messageQueue=MessageQueue [topic=ROCKETMQ_1, brokerName=huanghui-PF1L9MFC.al.com, queueId=0], queueOffset=0]
```

### msgId:[IP,PID,HASHCODE,时间差,计数]

#### MessageClientIDSetter#setUniqID

```java
public static void setUniqID(final Message msg) {
  if (msg.getProperty(MessageConst.PROPERTY_UNIQ_CLIENT_MESSAGE_ID_KEYIDX) == null) {
    msg.putProperty("UNIQ_KEY", createUniqID());
  }
}
↓
↓
public static String createUniqID() {
  StringBuilder sb = new StringBuilder(LEN * 2);
  sb.append(FIX_STRING);
  sb.append(UtilAll.bytes2string(createUniqIDBuffer()));
  return sb.toString();
}
↓
↓
//MessageClientIDSetter#static块
static {
  //10.0.192.122=[10, 0, -64, 122]
  byte[] ip = UtilAll.getIP();
  //16
  LEN = ip.length + 2 + 4 + 4 + 2;
  ByteBuffer tempBuffer = ByteBuffer.allocate(ip.length + 2 + 4);
  tempBuffer.put(ip);
  tempBuffer.putShort((short) UtilAll.getPid());
  tempBuffer.putInt(MessageClientIDSetter.class.getClassLoader().hashCode());
  //因子:IP,PID,HASHCODE
  FIX_STRING = UtilAll.bytes2string(tempBuffer.array());
  setStartTime(System.currentTimeMillis());
  COUNTER = new AtomicInteger(0);
}
↓
↓
private static byte[] createUniqIDBuffer() {
  ByteBuffer buffer = ByteBuffer.allocate(4 + 2);
  buffer.putInt((int) (System.currentTimeMillis() - startTime));
  buffer.putShort((short) COUNTER.getAndIncrement());
  //因子:时间差,计数
  return buffer.array();
}
```

#### MQClientAPIImpl#processSendResponse

```java
private SendResult processSendResponse(
  final String brokerName,
  final Message msg,
  final RemotingCommand response
)
  //解析响应结果
  SendMessageResponseHeader responseHeader = 
  (SendMessageResponseHeader) response.decodeCommandCustomHeader(SendMessageResponseHeader.class);

	//"UNIQ_KEY"
	String uniqMsgId = MessageClientIDSetter.getUniqID(msg);

	SendResult sendResult = new SendResult(sendStatus,
                                         uniqMsgId, //msgId
                                         responseHeader.getMsgId(), //offsetMsgId
                                         messageQueue,
                                         responseHeader.getQueueOffset()); //ConsumeQueue中的偏移量
}
```

### offsetMsgId:[Broker地址端口,消息在CommitLog中的偏移量]

#### 1.DefaultAppendMessageCallback#doAppend

```java
public AppendMessageResult doAppend(final long fileFromOffset,
                                    final ByteBuffer byteBuffer,
                                    final int maxBlank,
                                    final MessageExtBrokerInner msgInner) {
  //"8"
  int storeHostLength = (sysflag & MessageSysFlag.STOREHOSTADDRESS_V6_FLAG) == 0 ? 4 + 4 : 16 + 4;

  ByteBuffer storeHostHolder = ByteBuffer.allocate(storeHostLength);

  String msgId;
  if ((sysflag & MessageSysFlag.STOREHOSTADDRESS_V6_FLAG) == 0) {
    msgId = MessageDecoder.createMessageId(this.msgIdMemory,
                                           msgInner.getStoreHostBytes(storeHostHolder) //2
                                           , wroteOffset); //3
  }
}
```

#### 2.MessageExt#getStoreHostBytes

```java
public ByteBuffer getStoreHostBytes(ByteBuffer byteBuffer) {
  return socketAddress2ByteBuffer(this.storeHost, byteBuffer);
}
↓
↓
public static ByteBuffer socketAddress2ByteBuffer(final SocketAddress socketAddress,
                                                  final ByteBuffer byteBuffer) {
  //"/10.0.192.122:10911"
  InetSocketAddress inetSocketAddress = (InetSocketAddress) socketAddress;
  //"/10.0.192.122"
  InetAddress address = inetSocketAddress.getAddress();
  if (address instanceof Inet4Address) {
    //"[10, 0, -64, 122]"
    byteBuffer.put(inetSocketAddress.getAddress().getAddress(), 0, 4);
  }
  //"10911"
  byteBuffer.putInt(inetSocketAddress.getPort());
  byteBuffer.flip();
  return byteBuffer;
}

```

#### 3.MessageDecoder#createMessageId

```java
public static String createMessageId(final ByteBuffer input, final ByteBuffer addr, final long offset) {
	input.flip();
  int msgIDLength = addr.limit() == 8 ? 16 : 28;
  input.limit(msgIDLength);
  input.put(addr);
  input.putLong(offset);
  //因子:Broker地址端口,消息在CommitLog中的偏移量  
  return UtilAll.bytes2string(input.array());          
}
```

---

### 客户端消费消息

```
如果消息消费失败需要重试,此时RocketMQ会将消息重新发送到Broker,此时msgId不会发送变化
但该消息的offsetMsgId会发送变化,因为其存储在服务器中的位置发生了变化
```

```java
//MessageClientExt#getMsgId
public String getMsgId() {
    //如果存在就使用msgId返回,如果不存在就使用offsetMsgId返回
    String uniqID = MessageClientIDSetter.getUniqID(this);
    if (uniqID == null) {
        return this.getOffsetMsgId();
    } else {
        return uniqID;
    }
}

//MessageExt#toString
public String toString() {
  return "MessageExt [brokerName=" + brokerName + ", queueId=" + queueId + ", storeSize=" + storeSize + ", queueOffset=" + queueOffset + ", sysFlag=" + sysFlag + ", bornTimestamp=" + bornTimestamp + ", bornHost=" + bornHost + ", storeTimestamp=" + storeTimestamp + ", storeHost=" + storeHost
    //打印offsetMsgId
    + ", msgId=" + msgId
    + ", commitLogOffset=" + commitLogOffset + ", bodyCRC=" + bodyCRC + ", reconsumeTimes="+ reconsumeTimes + ", preparedTransactionOffset=" + preparedTransactionOffset + ", toString()=" + super.toString() + "]";
}
```

---

#### 控制台

**这个页面展示的Message ID是上面的msgId**

<img src="./MessageID_1.png" alt="Message ID" style="zoom:50%;" />

**这个页面可以通过msgId查询,可以通过offsetMsgId查询**

<img src="./MessageID_2.png" alt="Message Key查找" style="zoom:50%;" />
