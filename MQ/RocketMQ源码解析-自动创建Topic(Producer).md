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
    }
    //没有对应的Topic报错
    throw new MQClientException("No route info of this topic: " + msg.getTopic() + FAQUrl.suggestTodo(FAQUrl.NO_TOPIC_ROUTE_INFO),
            null).setResponseCode(ClientErrorCode.NOT_FOUND_TOPIC_EXCEPTION);
            
3.进入DefaultMQProducerImpl的tryToFindTopicPublishInfo
    
    //TopicPublishInfo保存本地Topic相关信息
    //默认会添加Topic:TBW102
    TopicPublishInfo topicPublishInfo = this.topicPublishInfoTable.get(topic);
    if (null == topicPublishInfo || !topicPublishInfo.ok()) {
        //从NameServer获取Topic的信息
        this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic);
        topicPublishInfo = this.topicPublishInfoTable.get(topic);
    }

    if (topicPublishInfo.isHaveTopicRouterInfo() || topicPublishInfo.ok()) {
        return topicPublishInfo;
    } else {
        //没获取到再次获取
        //参数跟上面不一样,使用Topic:TBW102
        this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic, true, this.defaultMQProducer);
        topicPublishInfo = this.topicPublishInfoTable.get(topic);
        return topicPublishInfo;
    }
    
4.进入MQClientInstance的updateTopicRouteInfoFromNameServer

    TopicRouteData topicRouteData;
    if (isDefault && defaultMQProducer != null) {
        //Topic:TBW102
        topicRouteData = this.mQClientAPIImpl.getDefaultTopicRouteInfoFromNameServer(defaultMQProducer.getCreateTopicKey(), 1000 * 3); //5
    } else {
        topicRouteData = this.mQClientAPIImpl.getTopicRouteInfoFromNameServer(topic, 1000 * 3);
    }
    ...
    //这里包含自己创建的Topic
    TopicPublishInfo publishInfo = topicRouteData2TopicPublishInfo(topic, topicRouteData);
    publishInfo.setHaveTopicRouterInfo(true);
    if (impl != null) {
        //更新topicPublishInfoTable
        impl.updateTopicPublishInfo(topic, publishInfo);
    }

5.进入MQClientAPIImpl的getDefaultTopicRouteInfoFromNameServer

    return getTopicRouteInfoFromNameServer(topic, timeoutMillis, false);
    ↓
    ↓
    //发送到NameServer
    RemotingCommand request = RemotingCommand.createRequestCommand(RequestCode.GET_ROUTEINFO_BY_TOPIC, requestHeader);
    RemotingCommand response = this.remotingClient.invokeSync(null, request, timeoutMillis);
    
---
    






