1.进入BrokerStartup的main

    BrokerController brokerController = createBrokerController(args); //2
    start(brokerController); //3

2.进入createBrokerController

    final BrokerController controller = new BrokerController(
                brokerConfig,
                nettyServerConfig,
                nettyClientConfig,
                messageStoreConfig);
    ↓
    ↓
    this.topicConfigManager = new TopicConfigManager(this);
    ↓
    ↓
    if (this.brokerController.getBrokerConfig().isAutoCreateTopicEnable()) {
        String topic = TopicValidator.AUTO_CREATE_TOPIC_KEY_TOPIC;
        TopicConfig topicConfig = new TopicConfig(topic);
        //TBW102
        this.topicConfigTable.put(topicConfig.getTopicName(), topicConfig);
    }

3.1进入BrokerStartup的start

    controller.start();

3.2进入BrokerController的start

    this.registerBrokerAll(true, false, true);
    ↓
    ↓
    TopicConfigSerializeWrapper topicConfigWrapper = this.getTopicConfigManager().buildTopicConfigSerializeWrapper();
    ...
    doRegisterBrokerAll(checkOrderConfig, oneway, topicConfigWrapper);
    ↓
    ↓
    //进入doRegisterBrokerAll
    List<RegisterBrokerResult> registerBrokerResultList = this.brokerOuterAPI.registerBrokerAll(
            this.brokerConfig.getBrokerClusterName(),
            this.getBrokerAddr(),
            this.brokerConfig.getBrokerName(),
            this.brokerConfig.getBrokerId(),
            this.getHAServerAddr(),
            topicConfigWrapper,
            this.filterServerManager.buildNewFilterServerList(),
            oneway,
            this.brokerConfig.getRegisterBrokerTimeoutMills(),
            this.brokerConfig.isCompressedRegister());

3.3进入BrokerOuterAPI的registerBrokerAll

    //构造上报给NameServer的数据
    RegisterBrokerBody requestBody = new RegisterBrokerBody();
    requestBody.setTopicConfigSerializeWrapper(topicConfigWrapper);
    
---

自定义的Topic消息到达Broker后  

1.进入AbstractSendMessageProcessor的msgCheck

    //获取不到
    TopicConfig topicConfig = this.brokerController.getTopicConfigManager().selectTopicConfig(requestHeader.getTopic());
    if (null == topicConfig) {
        topicConfig = this.brokerController.getTopicConfigManager().createTopicInSendMessageMethod(requestHeader.getTopic(), requestHeader.getDefaultTopic(), RemotingHelper.parseChannelRemoteAddr(ctx.channel()), requestHeader.getDefaultTopicQueueNums(), topicSysFlag);
    }

2.进入TopicConfigManager的createTopicInSendMessageMethod
  
    //创建自定义TopicConfig对象
    topicConfig = new TopicConfig(topic);
    ...
    if (topicConfig != null) {
        this.topicConfigTable.put(topic, topicConfig);
        //持久化到${user.home}/store/config/topics.json
        this.persist();
    }
    ...
    //同步到NameServer
    if (createNew) {
        this.brokerController.registerBrokerAll(false, true, true);
    }
    
---
    
