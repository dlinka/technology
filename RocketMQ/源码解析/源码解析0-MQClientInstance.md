```java
生产者和消费者的客户端公共代码
```

#### MQClientInstance

```java
public class MQClientInstance {
  private final ClientConfig clientConfig;
  //<Group, DefaultMQProducerImpl>
  private final ConcurrentMap<String, MQProducerInner> producerTable = new ConcurrentHashMap<>();
	//<Group, DefaultMQPushConsumerImpl>
  private final ConcurrentMap<String, MQConsumerInner> consumerTable = new ConcurrentHashMap<>();
	private final NettyClientConfig nettyClientConfig;
  private final MQClientAPIImpl mQClientAPIImpl;
  //<Topic, TopicRouteData>
  private final ConcurrentMap<String, TopicRouteData> topicRouteTable = new ConcurrentHashMap<>();
  //<BrokerName,<BrokerId, IP:PORT>>
  private final ConcurrentMap<String, HashMap<Long, String>> brokerAddrTable = new ConcurrentHashMap<>();
  private final ClientRemotingProcessor clientRemotingProcessor;
  private final PullMessageService pullMessageService;
  private final RebalanceService rebalanceService;
  
  public MQClientInstance(ClientConfig clientConfig,
                          int instanceIndex,
                          String clientId,
                          RPCHook rpcHook) {
    this.clientConfig = clientConfig;
    this.nettyClientConfig = new NettyClientConfig();
    this.clientRemotingProcessor = new ClientRemotingProcessor(this);
    this.mQClientAPIImpl = new MQClientAPIImpl(this.nettyClientConfig, this.clientRemotingProcessor, rpcHook, clientConfig);

    if (this.clientConfig.getNamesrvAddr() != null) {
      //"localhost:9876"
      this.mQClientAPIImpl.updateNameServerAddressList(this.clientConfig.getNamesrvAddr());
    }

    //消息消费
    this.pullMessageService = new PullMessageService(this);

    this.rebalanceService = new RebalanceService(this);
  }
}
```

#### MQClientAPIImpl

```java
public class MQClientAPIImpl {
	private final RemotingClient remotingClient;
  private final ClientRemotingProcessor clientRemotingProcessor;
	private ClientConfig clientConfig;
	
  public MQClientAPIImpl(final NettyClientConfig nettyClientConfig,
                         final ClientRemotingProcessor clientRemotingProcessor,
                         RPCHook rpcHook,
                         final ClientConfig clientConfig) {
    this.clientConfig = clientConfig;
    this.remotingClient = new NettyRemotingClient(nettyClientConfig, null);
    this.clientRemotingProcessor = clientRemotingProcessor;
  }
  
  public void updateNameServerAddressList(final String addrs) {
    //"localhost:9876"
  	this.remotingClient.updateNameServerAddressList(list);
  }
}
```

#### NettyRemotingClient

```java
public class NettyRemotingClient extends NettyRemotingAbstract implements RemotingClient {
	private final AtomicReference<List<String>> namesrvAddrList = new AtomicReference<List<String>>();
}
```

---