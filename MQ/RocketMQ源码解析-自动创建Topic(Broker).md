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

3.进入BrokerStartup的start

    controller.start();

2.进入BrokerController的start

    this.registerBrokerAll(true, false, true);
    ↓
    ↓
    
    TopicConfigSerializeWrapper topicConfigWrapper = this.getTopicConfigManager().buildTopicConfigSerializeWrapper();
    ...
    doRegisterBrokerAll(checkOrderConfig, oneway, topicConfigWrapper);
    ↓
    ↓
    
    
    
