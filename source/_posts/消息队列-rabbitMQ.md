---
title: 消息队列rabbitMQ的使用
date: 2018-11-01 22:06:30
tags: 
categories: 消息队列
---

消息队列中间件是分布式系统中重要的组件，主要解决应用耦合，异步消息，流量削锋等问题.

rbbitMQ是用Erlang语言实现的，它有几个概念：

```
broker：消息队列服务器实体。
exchange：消息交换机，它指定消息按什么规则，路由到哪个队列。
queue：消息队列，每个消息都会被投入到一个或多个队列。
binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
routing Key：路由关键字，exchange根据这个关键字进行消息投递。
vhost：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。
producer：消息生产者，就是投递消息的程序。
consumer：消息消费者，就是接受消息的程序。
channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。
```

消息队列的使用过程大概如下：

```
（1）客户端连接到消息队列服务器broker，打开一个channel。
（2）客户端声明一个exchange，并设置相关属性。
（3）客户端声明一个queue，并设置相关属性。
（4）客户端使用routing key，在exchange和queue之间建立好绑定关系。
（5）客户端投递消息到exchange。
（6）exchange接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里。
```

## 一.环境安装

1.由于RabbitMQ是用Erlang语言编写的，因此需要先安装Erlang。通过<http://www.erlang.org/downloads>获取对应安装文件进行安装,MQ下载地址<https://www.rabbitmq.com/download.html>

2.运行命令rabbitmq-plugins enable rabbitmq_management 开启Web管理插件

3.通过浏览器访问http://localhost:15672，并通过默认用户guest进行登录，密码也是guest，登录后的页面：

RabbitMQ中，所有生产者提交的消息都由Exchange来接受，然后Exchange按照特定的策略转发到Queue进行存储
RabbitMQ提供了四种Exchange：fanout,direct,topic,header

## 二.Exchange说明

### 一.DirectExchange

任何发送到Direct Exchange的消息都会被转发到RouteKey中指定的Queue。

1.一般情况可以使用rabbitMQ自带的Exchange：””(该Exchange的名字为空字符串，下文称其为default Exchange)。

2.这种模式下不需要将Exchange进行任何绑定(binding)操作

3.消息传递时需要一个“RouteKey”，可以简单的理解为要发送到的队列名字。

4.如果vhost中不存在RouteKey中指定的队列名，则该消息会被抛弃。

### 二.Fanout Exchange

任何发送到Fanout Exchange的消息都会被转发到与该Exchange绑定(Binding)的所有Queue上。

1.可以理解为路由表的模式

2.这种模式不需要RouteKey

3.这种模式需要提前将Exchange与Queue进行绑定，一个Exchange可以绑定多个Queue，一个Queue可以同多个Exchange进行绑定。

4.如果接受到消息的Exchange没有与任何Queue绑定，则消息会被抛弃。

### 三.Topic Exchange

任何发送到Topic Exchange的消息都会被转发到所有关心RouteKey中指定话题的Queue上

1.这种模式较为复杂，简单来说，就是每个队列都有其关心的主题，所有的消息都带有一个“标题”(RouteKey)，Exchange会将消息转发到所有关注主题能与RouteKey模糊匹配的队列。

2.这种模式需要RouteKey，也许要提前绑定Exchange与Queue。

3.在进行绑定时，要提供一个该队列关心的主题，如“#.log.#”表示该队列关心所有涉及log的消息(一个RouteKey为”MQ.log.error”的消息会被转发到该队列)。

4.“#”表示0个或若干个关键字，“”表示一个关键字。如“log.”能与“log.warn”匹配，无法与“log.warn.timeout”匹配；但是“log.#”能与上述两者匹配。

5.同样，如果Exchange没有发现能够与RouteKey匹配的Queue，则会抛弃此消息。

## 三springboot整合

### 一.导入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 二.配置文件

消息生产者

```
spring.application.name=springboot-rabbitmq
server.port=9511

spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.publisher-confirms=true
spring.rabbitmq.virtual-host=/
# 手动ACK 不开启自动ACK模式,目的是防止报错后未正确处理消息丢失 默认 为 none
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

消息消费者

```
spring.application.name=springboot-rabbitmq
server.port=9512


spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.publisher-confirms=true
spring.rabbitmq.virtual-host=/
# 手动ACK 不开启自动ACK模式,目的是防止报错后未正确处理消息丢失 默认 为 none
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

### 三.消息生产者

```java
package com.example.producer.mq;

import com.example.common.cons.MqType;
import com.example.common.model.Order;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.amqp.support.converter.AbstractJavaTypeMapper;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * @Author jiangyong
 * @Description TODO
 * @Date 2018/6/14 12:53
 **/
@Component
@Slf4j
public class OrderMqHandler {

    private AmqpTemplate amqpTemplate;


    public OrderMqHandler(AmqpTemplate amqpTemplate) {
        this.amqpTemplate = amqpTemplate;
    }

    public void sendGenerateOrder() {
        log.info("--------发送消息");
        Order order = new Order();
        order.setId(1);
        order.setAmount(100);
        order.setName("苹果手机");
        order.setOrderNo("20190910");
        amqpTemplate.convertAndSend(MqType.ORDER_QUEUE, order);
    }
     /**
     *发送延时消息
     **/
    public void sendDelay() {
        Order order = new Order();
        order.setId(1);
        order.setAmount(100);
        order.setName("苹果手机");
        order.setOrderNo("20190922");
        amqpTemplate.convertAndSend(MqType.REGISTER_DELAY_EXCHANGE, MqType.DELAY_ROUTING_KEY, order, message -> {
            // TODO 第一句是可要可不要,根据自己需要自行处理
            message.getMessageProperties().setHeader(AbstractJavaTypeMapper.DEFAULT_CONTENT_CLASSID_FIELD_NAME, Order.class.getName());
            // TODO 如果配置了 params.put("x-message-ttl", 5 * 1000); 那么这一句也可以省略,具体根据业务需要是声明 Queue 的时候就指定好延迟时间还是在发送自己控制时间
            message.getMessageProperties().setExpiration(5 * 1000 + "");
            return message;
        });
        log.info("发送消息时间:{}", LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    }

}
```

### 四.消息消费者

```java
package com.example.consumer.config;

import com.example.common.cons.MqType;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

/**
 * @Author jiangyong
 * @Description TODO
 * @Date 2018/6/14 14:39
 **/
@Configuration
@Slf4j
public class MqConfig {


    @Bean
    public Queue orderQueue() {
        // 第一个是 QUEUE 的名字,第二个是消息是否需要持久化处理(消息会持久化在磁盘中)
        return new Queue(MqType.ORDER_QUEUE, true);
    }

    /**
     * 延迟队列配置
     * <p>
     * 1、params.put("x-message-ttl", 5 * 1000);
     * TODO 第一种方式是直接设置 Queue 延迟时间 但如果直接给队列设置过期时间,这种做法不是很灵活,（当然二者是兼容的,默认是时间小的优先）
     * 2、rabbitTemplate.convertAndSend(book, message -> {
     * message.getMessageProperties().setExpiration(2 * 1000 + "");
     * return message;
     * });
     * TODO 第二种就是每次发送消息动态设置延迟时间,这样我们可以灵活控制,
     * TODO 在消息过期时,就会转发消息amqpTemplate.convertAndSend(exchange,routingKey,...)
     *
     **/
    @Bean
    public Queue delayProcessQueue() {
        Map<String, Object> params = new HashMap<>();
        // x-dead-letter-exchange 声明了队列里的死信转发到的DLX名称，
        params.put("x-dead-letter-exchange", MqType.REGISTER_EXCHANGE_NAME);
        // x-dead-letter-routing-key 声明了这些死信在转发时携带的 routing-key 名称。
        params.put("x-dead-letter-routing-key", MqType.ROUTING_KEY);
        return new Queue(MqType.REGISTER_DELAY_QUEUE, true, false, false, params);
    }

    /**
     * 需要将一个队列绑定到交换机上，要求该消息与一个特定的路由键完全匹配。
     * 这是一个完整的匹配。如果一个队列绑定到该交换机上要求路由键 “dog”，则只有被标记为“dog”的消息才被转发，不会转发dog.puppy，也不会转发dog.guard，只会转发dog。
     * TODO 它不像 TopicExchange 那样可以使用通配符适配多个
     *
     * @return DirectExchange
     */
    @Bean
    public DirectExchange delayExchange() {
        return new DirectExchange(MqType.REGISTER_DELAY_EXCHANGE);
    }

    @Bean
    public Binding dlxBinding() {
        return BindingBuilder.bind(delayProcessQueue()).to(delayExchange()).with(MqType.DELAY_ROUTING_KEY);
    }


    @Bean
    public Queue registerOrderQueue() {
        return new Queue(MqType.REGISTER_QUEUE_NAME, true);
    }

    /**
     * 将路由键和某模式进行匹配。此时队列需要绑定要一个模式上。
     * 符号“#”匹配一个或多个词，符号“*”匹配不多不少一个词。因此“audit.#”能够匹配到“audit.irs.corporate”，但是“audit.*” 只会匹配到“audit.irs”。
     **/
    @Bean
    public TopicExchange registerOrderTopicExchange() {
        return new TopicExchange(MqType.REGISTER_EXCHANGE_NAME);
    }

    @Bean
    public Binding registerBookBinding() {
        // TODO 如果要让延迟队列之间有关联,这里的 routingKey 和 绑定的交换机很关键
        return BindingBuilder.bind(registerOrderQueue()).to(registerOrderTopicExchange()).with(MqType.ROUTING_KEY);
    }

}
```

```java
package com.example.consumer.mq;

import com.example.common.cons.MqType;
import com.example.common.model.Order;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * @Author jiangyong
 * @Description TODO
 * @Date 2018/6/14 12:59
 **/
@Slf4j
@Component
public class OrderMqListen {

    @RabbitListener(queues = {MqType.ORDER_QUEUE})
    public void process(Order order, Message message, Channel channel) {
        try {
            long deliveryTag = message.getMessageProperties().getDeliveryTag();
            // TODO 通知 MQ 消息已被成功消费,可以ACK了
            channel.basicAck(deliveryTag, false);
            log.info("receive message:{}", order);
        } catch (Exception e) {
            try {
                // TODO 处理失败,重新压入MQ
                channel.basicRecover();
            } catch (IOException e1) {
                e1.printStackTrace();
            }
        }

    }

    @RabbitListener(queues = {MqType.REGISTER_QUEUE_NAME})
    public void delayProcess(Order order, Message message, Channel channel) {
        log.info("order:{}", order);
        log.info("接收时间为:{}", LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        try {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (IOException e) {
            //TODO 失败处理
        }
    }
}
```