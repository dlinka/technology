1.进入DefaultMQProducer的send

    return this.defaultMQProducerImpl.send(msg);

2.进入DefaultMQProducerImpl的send

    return send(msg, this.defaultMQProducer.getSendMsgTimeout());
    ↓
    ↓
    return this.sendDefaultImpl(msg, CommunicationMode.SYNC, null, timeout);
    ↓
    ↓
    TopicPublishInfo topicPublishInfo = this.tryToFindTopicPublishInfo(msg.getTopic());
    if (topicPublishInfo != null && topicPublishInfo.ok()) {
        //这里就是获取到了发送的逻辑
    }
    //这里没有对应的Topic就报错
    throw new MQClientException("No route info of this topic: " + msg.getTopic() + FAQUrl.suggestTodo(FAQUrl.NO_TOPIC_ROUTE_INFO),
            null).setResponseCode(ClientErrorCode.NOT_FOUND_TOPIC_EXCEPTION);
            
3.进入DefaultMQProducerImpl的tryToFindTopicPublishInfo
    //TopicPublishInfo保存本地Topic相关信息
    //启动的时候会添加Topic:TBW102
    TopicPublishInfo topicPublishInfo = this.topicPublishInfoTable.get(topic);
    if (null == topicPublishInfo || !topicPublishInfo.ok()) {
        //从NameServer获取Topic的信息
        this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic);
        topicPublishInfo = this.topicPublishInfoTable.get(topic);
    }

    //获取到了返回
    if (topicPublishInfo.isHaveTopicRouterInfo() || topicPublishInfo.ok()) {
        return topicPublishInfo;
    } else {
        //没获取到再次获取
        //参数不一样
        this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic, true, this.defaultMQProducer);
        topicPublishInfo = this.topicPublishInfoTable.get(topic);
        return topicPublishInfo;
    }
    
4.进入MQClientInstance的updateTopicRouteInfoFromNameServer

    return updateTopicRouteInfoFromNameServer(topic, false, null);
    ↓
    ↓
    TopicRouteData topicRouteData;
    if (isDefault && defaultMQProducer != null) {
        //获取Topic:TBW102
        topicRouteData = this.mQClientAPIImpl.getDefaultTopicRouteInfoFromNameServer(defaultMQProducer.getCreateTopicKey(), 1000 * 3);
    } else {
        topicRouteData = this.mQClientAPIImpl.getTopicRouteInfoFromNameServer(topic, 1000 * 3);
    }

5.进入MQClientAPIImpl的getDefaultTopicRouteInfoFromNameServer

    return getTopicRouteInfoFromNameServer(topic, timeoutMillis, false);
    ↓
    ↓
    //这就是发送NameServer的代码
    RemotingCommand request = RemotingCommand.createRequestCommand(RequestCode.GET_ROUTEINFO_BY_TOPIC, requestHeader);
    RemotingCommand response = this.remotingClient.invokeSync(null, request, timeoutMillis);
    
---
    






