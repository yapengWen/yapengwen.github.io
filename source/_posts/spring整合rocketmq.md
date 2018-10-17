---
title: Spring整合RocketMQ
date: 2017-12-15 13:00:00
tags: RocketMQ
categories: MQ
---
##### Spring 整合 RocketMQ
1. 引入jar包

```
<!-- RocketMQ -->
        <dependency>  
             <groupId>com.alibaba.rocketmq</groupId>  
             <artifactId>rocketmq-all</artifactId>  
             <version>3.2.6</version>  
             <type>pom</type>  
        </dependency>
        <dependency>  
             <groupId>com.alibaba.rocketmq</groupId>  
             <artifactId>rocketmq-client</artifactId>  
             <version>3.2.6</version>  
        </dependency> 
```


2.Spring bean 配置单例

```
    <bean id="myProducer" class="cn.zno.rocketmq.MyProducer"
            init-method="init"  
            destroy-method="destroy"
            scope="singleton">
　　　　　　<property name="producerGroup" value="MyProducerGroup" />
　　　　　　<property name="namesrvAddr" value="127.0.0.1:9876" />
    </bean>
    <bean class="cn.zno.rocketmq.MyConsumer" 
            init-method="init" 
            destroy-method="destroy"
            scope="singleton">
　　　　　　<property name="consumerGroup" value="MyConsumerGroup" />
　　　　　　<property name="namesrvAddr" value="127.0.0.1:9876" />
    </bean> 
```

3. 自定义producer

```
package cn.zno.rocketmq;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;

public class MyProducer {

    private final Logger logger = LoggerFactory.getLogger(MyProducer.class);

    private DefaultMQProducer defaultMQProducer;
    private String producerGroup;
    private String namesrvAddr;

    /**
     * Spring bean init-method
     */
    public void init() throws MQClientException {
        // 参数信息
        logger.info("DefaultMQProducer initialize!");
        logger.info(producerGroup);
        logger.info(namesrvAddr);

        // 初始化
        defaultMQProducer = new DefaultMQProducer(producerGroup);
        defaultMQProducer.setNamesrvAddr(namesrvAddr);
        defaultMQProducer.setInstanceName(String.valueOf(System.currentTimeMillis()));
        
        defaultMQProducer.start();

　　　　 logger.info("DefaultMQProudcer start success!");

    }

    /**
     * Spring bean destroy-method
     */
    public void destroy() {
        defaultMQProducer.shutdown();
    }

    public DefaultMQProducer getDefaultMQProducer() {
        return defaultMQProducer;
    }

    // ---------------setter -----------------

    public void setProducerGroup(String producerGroup) {
        this.producerGroup = producerGroup;
    }

    public void setNamesrvAddr(String namesrvAddr) {
        this.namesrvAddr = namesrvAddr;
    }

}
```


4. 自定义consumer

```
package cn.zno.rocketmq;

import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;
import com.alibaba.rocketmq.common.message.MessageExt;
import com.alibaba.rocketmq.common.protocol.heartbeat.MessageModel;

public class MyConsumer {

    private final Logger logger = LoggerFactory.getLogger(MyConsumer.class);

    private DefaultMQPushConsumer defaultMQPushConsumer;
    private String namesrvAddr;
    private String consumerGroup;

    /**
     * Spring bean init-method
     */
    public void init() throws InterruptedException, MQClientException {

        // 参数信息
        logger.info("DefaultMQPushConsumer initialize!");
        logger.info(consumerGroup);
        logger.info(namesrvAddr);

        // 一个应用创建一个Consumer，由应用来维护此对象，可以设置为全局对象或者单例<br>
        // 注意：ConsumerGroupName需要由应用来保证唯一
        defaultMQPushConsumer = new DefaultMQPushConsumer(consumerGroup);
        defaultMQPushConsumer.setNamesrvAddr(namesrvAddr);
        defaultMQPushConsumer.setInstanceName(String.valueOf(System.currentTimeMillis()));

        // 订阅指定MyTopic下tags等于MyTag

        defaultMQPushConsumer.subscribe("MyTopic", "MyTag");

        // 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费<br>
        // 如果非第一次启动，那么按照上次消费的位置继续消费
        defaultMQPushConsumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        // 设置为集群消费(区别于广播消费)
        defaultMQPushConsumer.setMessageModel(MessageModel.CLUSTERING);

        defaultMQPushConsumer.registerMessageListener(new MessageListenerConcurrently() {

            // 默认msgs里只有一条消息，可以通过设置consumeMessageBatchMaxSize参数来批量接收消息
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {

                MessageExt msg = msgs.get(0);
                if (msg.getTopic().equals("MyTopic")) {
                    // TODO 执行Topic的消费逻辑
                    if (msg.getTags() != null && msg.getTags().equals("MyTag")) {
                        // TODO 执行Tag的消费
                    }
                }
                // 如果没有return success ，consumer会重新消费该消息，直到return success
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        // Consumer对象在使用之前必须要调用start初始化，初始化一次即可<br>
        defaultMQPushConsumer.start();

        logger.info("DefaultMQPushConsumer start success!");
    }

    /**
     * Spring bean destroy-method
     */
    public void destroy() {
        defaultMQPushConsumer.shutdown();
    }

    // ----------------- setter --------------------

    public void setNamesrvAddr(String namesrvAddr) {
        this.namesrvAddr = namesrvAddr;
    }

    public void setConsumerGroup(String consumerGroup) {
        this.consumerGroup = consumerGroup;
    }

}
```

5. 发消息

```
@Autowired
    private MyProducer myProducer;

    public void sendMessage() {
        Message msg = new Message("MyTopic", "MyTag", (JSONObject.fromObject(someMessage)).getBytes());
        SendResult sendResult = null;
        try {
            sendResult = myProducer.getDefaultMQProducer().send(msg);
        } catch (MQClientException e) {
            logger.error(e.getMessage() + String.valueOf(sendResult));
        }
        // 当消息发送失败时如何处理
        if (sendResult == null || sendResult.getSendStatus() != SendStatus.SEND_OK) {
            // TODO
        }
    }
```
