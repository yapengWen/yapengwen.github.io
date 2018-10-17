---
title: 关于RabbitMQ以及RabbitMQ和Spring的整合
date: 2017-12-15 17:00:00
tags: [RabbitMQ,spring]
categories: MQ
---

##### 一、maven依赖


```
<!--rabbitmq依赖 -->  
        <dependency>  
            <groupId>org.springframework.amqp</groupId>  
            <artifactId>spring-rabbit</artifactId>  
            <version>1.3.5.RELEASE</version>  
        </dependency>  
```

##### 二、消费端spring-rabbitmq.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" xmlns:rabbit="http://www.springframework.org/schema/rabbit"
	xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.0.xsd
    http://www.springframework.org/schema/rabbit  
    http://www.springframework.org/schema/rabbit/spring-rabbit-1.2.xsd">

	<!--配置connection-factory，指定连接rabbit server参数 -->
	<rabbit:connection-factory id="connectionFactory"
		virtual-host="${rabbitmq.virtualhost}" username="${rabbitmq.username}"
		password="${rabbitmq.password}" host="${rabbitmq.host}" port="${rabbitmq.port}" />

	<!--通过指定下面的admin信息，当前producer中的exchange和queue会在rabbitmq服务器上自动生成 -->
	<rabbit:admin id="connectAdmin" connection-factory="connectionFactory" />

	<!--定义queue -->
	<rabbit:queue name="${rabbitmq.queue}" durable="true" auto-delete="false"
		exclusive="false" declared-by="connectAdmin" />

	<!-- 定义direct exchange，绑定queueTest -->
	<!-- <rabbit:direct-exchange name="${rabbitmq.direct.exchange}"
		durable="true" auto-delete="false" declared-by="connectAdmin">
		<rabbit:bindings>
			<rabbit:binding queue="${rabbitmq.queue}" key="${rabbitmq.queue.queueKey}"></rabbit:binding>
		</rabbit:bindings>
	</rabbit:direct-exchange> -->

	<!--定义rabbit template用于数据的接收和发送 -->
	<!-- <rabbit:template id="amqpTemplate" connection-factory="connectionFactory"
		exchange="${rabbitmq.direct.exchange}" /> -->

	<!-- 消息接收者 -->
	<bean id="messageReceiver"
		class="com.showshine.pullq.rabbitmq.OfflineRabbitMessageListener"></bean>

	<!-- queue litener 观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象 -->
	<rabbit:listener-container
		connection-factory="connectionFactory">
		<rabbit:listener queues="${rabbitmq.queue}" ref="messageReceiver" />
	</rabbit:listener-container>

</beans>
```
##### 三、生产spring-rabbitmq.xml


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:rabbit="http://www.springframework.org/schema/rabbit"
	xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.0.xsd
    http://www.springframework.org/schema/rabbit  
    http://www.springframework.org/schema/rabbit/spring-rabbit-1.2.xsd">

	<!--配置connection-factory，指定连接rabbit server参数 -->
    <rabbit:connection-factory id="connectionFactory" virtual-host="virtualTest" 
        username="user" password="user" host="192.168.12.210" port="5672" 
        />

    <!--通过指定下面的admin信息，当前producer中的exchange和queue会在rabbitmq服务器上自动生成 -->
    <rabbit:admin id="connectAdmin" connection-factory="connectionFactory" />

    <!-- 定义direct exchange，绑定queueTest -->
    <rabbit:direct-exchange name="rabbitmqexch"
        durable="true" auto-delete="false" declared-by="connectAdmin">
    </rabbit:direct-exchange>

    <!--定义rabbit template用于数据的接收和发送 -->
    <rabbit:template id="amqpTemplate" connection-factory="connectionFactory"
        exchange="rabbitmqexch" />


</beans>
```
##### 四、rabbitmq.properties

```
rabbitmq.virtualhost=virtualTest
rabbitmq.username=user
rabbitmq.password=user
rabbitmq.host=127.0.0.1
rabbitmq.port=5672
rabbitmq.queue=rabbitque
rabbitmq.queue.queueKey=queueTestKey
rabbitmq.direct.exchange=rabbitmqexch
```
五、接收端RabbitMessageListener监听类

```
package com.showshine.pullq.rabbitmq;

import org.apache.log4j.Logger;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageListener;


/**
 * <dl>
 * <dt><b>类功能概要：</b></dt>
 * <dd>信贷业务：rocketmq消费端-线下转线上</dd>
 * <dd></dd>
 * </dl>
 * 
 * @copyright Copyright 2017,(C)The Trustrock Co.Ltd. All right reserved.
 * @since 1.6
 * @version <pre>
 * Version  Date                          Company    Author         Case-Name
 * -------  ------------------------      ---------  -------------  --------------
 * 1.00     2017年12月15日 上午11:54:23     Trustrock  wenyapeng      RocketMessageListener 
 * 
 * </pre>
 */
public class OfflineRabbitMessageListener implements MessageListener{
	
	private static Logger logger = Logger.getLogger(OfflineRabbitMessageListener.class);


	@Override
    public void onMessage(Message message) {
         logger.info("consumer receive message------->:{}" + message);  
        
    }

	
}

```

##### 六、发送端


```
package com.aliyun.openservices.client;

import org.junit.Before;
import org.junit.Test;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class RabbitmqTest {

	private AmqpTemplate amqpTemplate;

	private ApplicationContext context = null;

	@Before
	public void setUp() throws Exception {
		context = new ClassPathXmlApplicationContext("spring-rabbitmq.xml");
	}

	@Test
	public void should_send_a_amq_message() throws Exception {
		amqpTemplate = (AmqpTemplate) context.getBean("amqpTemplate");
		int a = 2;
		while (a > 0) {
			amqpTemplate.convertAndSend("queueTestKey", "wqwqw" + a--);
			try {
				// 暂停一下，好让消息消费者去取消息打印出来
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

		}
	}

}

```


[参考https://www.cnblogs.com/s648667069/p/6401463.html](https://www.cnblogs.com/s648667069/p/6401463.html)