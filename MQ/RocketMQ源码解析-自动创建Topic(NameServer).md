1.进入DefaultRequestProcessor的processRequest

    //注册Broker到NameServer
    case RequestCode.REGISTER_BROKER:
        Version brokerVersion = MQVersion.value2Version(request.getVersion());
        if (brokerVersion.ordinal() >= MQVersion.Version.V3_0_11.ordinal()) {
            return this.registerBrokerWithFilterServer(ctx, request); //2
        }

    //从NameServer获取Topic信息
    case RequestCode.GET_ROUTEINFO_BY_TOPIC:
        return this.getRouteInfoByTopic(ctx, request); //3

2.1进入DefaultRequestProcessor的registerBrokerWithFilterServer

    RegisterBrokerResult result = this.namesrvController.getRouteInfoManager().registerBroker(
            requestHeader.getClusterName(),
            requestHeader.getBrokerAddr(),
            requestHeader.getBrokerName(),
            requestHeader.getBrokerId(),
            requestHeader.getHaServerAddr(),
            //这个参数的详情看Broker篇
            registerBrokerBody.getTopicConfigSerializeWrapper(),
            registerBrokerBody.getFilterServerList(),
            ctx.channel());

2.2进入RouteInfoManager的registerBroker

     this.createAndUpdateQueueData(brokerName, entry.getValue());
     ↓
     ↓
     List<QueueData> queueDataList = this.topicQueueTable.get(topicConfig.getTopicName());
     if (null == queueDataList) {
         queueDataList = new LinkedList<QueueData>();
         queueDataList.add(queueData);
         //这里保存Broker的Topic:TBW102
         this.topicQueueTable.put(topicConfig.getTopicName(), queueDataList);
     }

3.1进入DefaultRequestProcessor的getRouteInfoByTopic

    TopicRouteData topicRouteData = this.namesrvController.getRouteInfoManager().pickupTopicRouteData(requestHeader.getTopic());
     
3.2进入RouteInfoManager的pickupTopicRouteData

    List<QueueData> queueDataList = this.topicQueueTable.get(topic);
    //这里可以获取到Topic:TBW102
    if (queueDataList != null) {
        topicRouteData.setQueueDatas(queueDataList);
        foundQueueData = true;
    }

---
