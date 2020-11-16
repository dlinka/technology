1.DefaultMQProducer#send

    this.defaultMQProducerImpl.send(msg);
    ↓
    ↓
    return send(msg, this.defaultMQProducer.getSendMsgTimeout());
    ↓
    ↓
    return this.sendDefaultImpl(msg, CommunicationMode.SYNC, null, timeout);
    ↓
    ↓
    TopicPublishInfo topicPublishInfo = this.tryToFindTopicPublishInfo(msg.getTopic()); //2
    ...
    MessageQueue mqSelected = this.selectOneMessageQueue(topicPublishInfo, lastBrokerName); //3
    ...
    sendResult = this.sendKernelImpl(msg, mq, communicationMode, sendCallback, topicPublishInfo, timeout - costTime);
    
2.DefaultMQProducerImpl#tryToFindTopicPublishInfo

    this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic);
    ↓
    ↓
    return updateTopicRouteInfoFromNameServer(topic, false, null);
    ↓
    ↓
    TopicPublishInfo publishInfo = topicRouteData2TopicPublishInfo(topic, topicRouteData);
    ↓
    ↓
    TopicPublishInfo info = new TopicPublishInfo();
    ...
    MessageQueue mq = new MessageQueue(topic, qd.getBrokerName(), i);
    info.getMessageQueueList().add(mq);
    
3.DefaultMQProducerImpl#selectOneMessageQueue

    return this.mqFaultStrategy.selectOneMessageQueue(tpInfo, lastBrokerName);
    ↓
    ↓
    return tpInfo.selectOneMessageQueue(lastBrokerName);
    ↓
    ↓
    if (lastBrokerName == null) {
        return selectOneMessageQueue();
    }
    ↓
    ↓
    int index = this.sendWhichQueue.getAndIncrement();
    int pos = Math.abs(index) % this.messageQueueList.size();
    //
    return this.messageQueueList.get(pos);
    
    
    
    
    
    
    
    
    
    
