Demo

    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer(CONSUMER_GROUP);
    consumer.setNamesrvAddr("localhost:9876");
    consumer.subscribe(TOPIC, "*");
    consumer.registerMessageListener(new MessageListenerConcurrently() {
        @Override
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
    });
    consumer.start();
    
1.DefaultMQPushConsumer#start

    this.defaultMQPushConsumerImpl.start();
    ↓
    ↓
    case CREATE_JUST:
        else if (this.getMessageListenerInner() instanceof MessageListenerConcurrently) {
            //消费消息
            this.consumeMessageService = new ConsumeMessageConcurrentlyService(this, (MessageListenerConcurrently) this.getMessageListenerInner());
        }
        ...
        mQClientFactory.start();
    ↓
    ↓
    this.pullMessageService.start();

2.PullMessageService#run

    this.pullMessage(pullRequest);
    ↓
    ↓
    DefaultMQPushConsumerImpl impl = (DefaultMQPushConsumerImpl) consumer;
    impl.pullMessage(pullRequest);
    ↓
    ↓
    PullCallback pullCallback = new PullCallback() {
        @Override
        public void onSuccess(PullResult pullResult) {
            case FOUND:
                //消费消息
                DefaultMQPushConsumerImpl.this.consumeMessageService.submitConsumeRequest(
                                    pullResult.getMsgFoundList(),
                                    processQueue,
                                    pullRequest.getMessageQueue(),
                                    dispatchToConsume);
        }
    };
    ...
    this.pullAPIWrapper.pullKernelImpl(
                pullRequest.getMessageQueue(),
                subExpression,
                subscriptionData.getExpressionType(),
                subscriptionData.getSubVersion(),
                pullRequest.getNextOffset(),
                this.defaultMQPushConsumer.getPullBatchSize(),
                sysFlag,
                commitOffsetValue,
                BROKER_SUSPEND_MAX_TIME_MILLIS,
                CONSUMER_TIMEOUT_MILLIS_WHEN_SUSPEND,
                CommunicationMode.ASYNC,
                pullCallback
            );
    ↓
    ↓
    PullResult pullResult = this.mQClientFactory.getMQClientAPIImpl().pullMessage(
                brokerAddr,
                requestHeader,
                timeoutMillis,
                communicationMode,
                pullCallback);
    ↓
    ↓
    case ASYNC:
        this.pullMessageAsync(addr, request, timeoutMillis, pullCallback);
    ↓
    ↓
    pullCallback.onSuccess(pullResult);
    
---
