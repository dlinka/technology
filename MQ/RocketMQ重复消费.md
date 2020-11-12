Demo

    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer(CONSUMER_GROUP);
    consumer.setNamesrvAddr("localhost:9876");
    consumer.subscribe(TOPIC, "*");
    //下面两个参数只会有一个线程处理消息
    consumer.setConsumeThreadMax(1);
    consumer.setConsumeThreadMin(1);
    //最多一次拉取30条
    consumer.setPullBatchSize(30);
    //回调函数一次只处理一个消息
    consumer.setConsumeMessageBatchMaxSize(1);
    consumer.registerMessageListener(new MessageListenerConcurrently() {
        @Override
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
    });
    consumer.start();
    
背景

    生产者发送200个消息
    开启4个消费者
    
