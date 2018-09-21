---

layout: post
title: rabbitmq--整合springBoot
date: 2018-06-28 01:46:04
tags: rabbitmq
categories: rabbitmq,springBoot
---

### 基础知识

#### 1、队列、生产者、消费者

队列（queue）是RabbitMQ的内部对象，用于存储消息。
生产者（proverder ），生产消息并投递到队列中。
消费者（consumer）可以从队列中获取消息并消费。如果有多个消费者，队列中的消息会平均分摊给各个消费者，而不是每个消费者能获取所有的队列消息并处理。

实际情况，一般是生产者绑定Exchange（交换机），通过指定一个Binding key，将Exchange与Queue绑定关联起来。
于是，生产者可以通过Routing key指定具体的路由规则，决定消息流向那个Queue中。Bindingkey与Rounting key一致才能取出该队列。

#### 2、Exchange交换机
RabbitMQ常用的Exchange Type有四种：fanout、direct、topic、headers 。

- direct：(默认方式)创建消息队列的时候,指定一个BindingKey。当发送者发送消息的时候,指定对应的Key。当Key和消息队列的BindingKey一致的时候,消息将会被发送到该消息队列中。先匹配，在投送。

- topic：转发信息主要是依据通配符（`.`和`*`），队列和交换机的绑定主要是依据一种模式(通配符+字符串)，而当发送消息的时候，只有指定的Key和该模式相匹配的时候，消息才会被发送到该消息队列中。最为灵活的方式。

- headers：队列和交换机的绑定的通过指定的一组键值对，而发送消息的时候也会指定一组键值对规则，当两组键值对规则相匹配的时候，消息会被发送到匹配的消息队列中。

- fanout：把所有发送到该Exchange的消息投递到所有与它绑定的队列中，设置的任何Key此时都无效。消息广播方式。

“quick.orange.rabbit”指定了两个队列 *.orange和 *.rabbit

<!-- more -->


#### 特别注意点

1、如果队列不存在，那么生产者发送的消息被rabbitmq接收到后，会自动被丢弃。同时，对应的消费者也接收不到消息。
为了数据不丢失，rabbitmq规定消费者和生产者都可以创建队列，但是以第一次创建的为准。

2、默认情况下，如果消息已经被某个消费者正确的接收到了，那么该消息就会被从队列中移除。
如果一个队列没有消费者，那么，如果这个队列有数据到达，那么这个数据会被缓存，不会被丢弃。当有消费者时，这个数据会被立即发送到这个消费者

3、ACK机制：可以显示的在程序中进行Ack，也可以自动的进行Ack。
如果某个消费者有bug，忘记了ack，那么RabbitMQServer会一直发送数据给它，因为Server认为这个消费者处理能力有限。当然你也可以发送Nack，让
让服务器继续发消息，直到服务器收到Ack才会将该消息删除。


### speingBoot中 使用 RabbitMQ
1、添加依赖：
```xml
 <!-- 添加springboot对amqp的支持 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
2、添加配置文件：
```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
```
3、定义配置：

```java
@Configuration
@Slf4j
public class RabbitMQConfig{

    public final static String QUEUE_NAME = "email";
    public final static String EXCHANGE_NAME = "exchange";
    public final static String ROUTING_KEY = "key";

    @Value("${spring.rabbitmq.host}")
    private String host;

    @Value("${spring.rabbitmq.port}")
    private String port;

    @Value("${spring.rabbitmq.username}")
    private String username;

    @Value("${spring.rabbitmq.password}")
    private String password;

    /** 创建队列 */
    @Bean
    public Queue queue() {
        return new Queue(QUEUE_NAME);
    }

    /** 创建一个 topic 类型的交换器 */
    @Bean
    public TopicExchange exchange() {
        return new TopicExchange(EXCHANGE_NAME);
    }

    /** 使用路由键（routingKey）把队列（Queue）绑定到交换器（Exchange）*/
    @Bean
    public Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(ROUTING_KEY);
    }

    /** 以下代码用于获取一个对象 */
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {

        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(new Jackson2JsonMessageConverter());
        return template;
    }

    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory(host, Integer.parseInt(port));
        connectionFactory.setUsername(username);
        connectionFactory.setPassword(password);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setPublisherConfirms(true);
        connectionFactory.addConnectionListener(new ConnectionListener() {
            @Override
            public void onCreate(Connection connection) {
                logger.info("已连接到rabbitmq服务器！");
            }
            @Override
            public void onClose(Connection connection) {
                logger.warn("无法连接到rabbitmq服务器！");
            }
        });
        return connectionFactory;
    }

    /** 初次启动执行 */
    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setMessageConverter(new Jackson2JsonMessageConverter());
        factory.setAcknowledgeMode(AcknowledgeMode.AUTO);
        return factory;
    }

    /** 覆写onMessage。即使收到消息后，执行报错，也不会循环去获取消息队列里的内容*/
    @Bean
    public SimpleMessageListenerContainer messageContainer() {

        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory());
        /** 绑定队列，不然不会执行*/
        container.setQueues(queue());
        container.setExposeListenerChannel(true);
        container.setMaxConcurrentConsumers(1);
        container.setConcurrentConsumers(1);
        /** 设置确认模式自动 */
        container.setAcknowledgeMode(AcknowledgeMode.AUTO);

//        container.setMessageListener(new ChannelAwareMessageListener() {
//            @Override
//            public void onMessage(Message message, Channel channel) throws Exception {
//                logger.info("message="+message+",channel="+channel);
//                try{
//                    byte[] body = message.getBody();
//                    if(body.equals("")){
//                        throw new NullPointerException();
//                    }
//                    /** 判断服务是否可用 */
//                    imFeign.isOk();
//                    /** 确认消息成功接受,不然会服务器会一直给推送*/
//                    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
//                }catch(Exception e){
//                    logger.warn("启动异常："+e);
//                    /** 服务不可用时，不消费，需要一直推送*/
//                    //channel.basicNack(message.getMessageProperties().getDeliveryTag(),false,true);
//
//                    Map<String, Object> arguments = new HashMap<String, Object>();
//                    arguments.put("x-dead-letter-exchange", EXCHANGE_NAME);
//                    arguments.put("x-message-ttl", 30 * 1000);
//
//                    channel.queueDeclare(QUEUE_NAME , true, false, false, arguments);
//           void BasicReject(message.getMessageProperties(), false);
//                }
//            }
//        });
        return container;
    }
}
```

4、创建生产者（以topic方式为例）：

```java
##先配置队列，在配置交换机
public class SenderConf {

   @Bean(name="message")
   public Queue queueMessage() {
      return new Queue("topic.message");
   }

   @Bean(name="messages")
   public Queue queueMessages() {
      return new Queue("topic.messages");
   }

   @Bean
   public TopicExchange exchange() {
      return new TopicExchange("exchange");//配置topic路由
   }

   //将topic.message与exchange绑定，binding_key为topic.message的完全匹配
   @Bean
   Binding bindingExchangeMessage(@Qualifier("message") Queue queueMessage, TopicExchange exchange{
       return BindingBuilder.bind(queueMessage).to(exchange).with("topic.message");
   }

   //将topic.message与exchange绑定，binding_key为topic.#的模糊匹配
   @Bean
   Binding bindingExchangeMessages(@Qualifier("messages") Queue queueMessages, TopicExchange exchange){
      return BindingBuilder.bind(queueMessages).to(exchange).with("topic.#");//*表示一个词,#表示零个或多个词
        }
}
```

5、创建消费者：

```java
@RabbitListener(queues="topic.message")    //监听器监听指定的Queue
public void process1(String str) {    
    System.out.println("message:"+str);
}

@RabbitListener(queues="topic.messages")    //监听器监听指定的Queue
public void process2(String str) {
    System.out.println("messages:"+str);
}
```
6、发送一条消息测试：

```java
template.converAndSend("exchange","topic.message","hello!");
```
>方法的第一个参数是交换机名称,第二个参数是发送的key,第三个参数是内容,RabbitMQ将会根据第二个参数去寻找有没有匹配此规则的队列,如果有,则把消息给它,如果有不止一个,则把消息分发给匹配的队列(每个队列都有消息!),在上面的测试代码中,参数2匹配了两个队列,因此消息将会被发放到这两个队列中，当然两个消费者也都能收到消息。

### 采坑

#### 接收不了对象

1、传输String类型的数据格式都没有问题，但是传递Object对象，就是接收不到或者无法解析.
>解决办法：改写RabbitTemplate，在配置文件中加入如下代码：

```java
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {

        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(new Jackson2JsonMessageConverter());
        return template;
    }
    
    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setMessageConverter(new Jackson2JsonMessageConverter());
        return factory;
    }
```

#### rabbitmq获取不到数据

>解决办法：查看是否有队列产生，交换机是否绑定了队列。参考：[文献](https://blog.csdn.net/li_101357/article/details/52853730)

rabbitmq有一个匿名交换机：`Exchange: (AMQP default)`，如果不指定默认会采用这个交换机进行交互。
因为rabbitmq必须要有交换机才能完成消息的发布与订阅（可以没有队列，但是必须要有交换机，当然，没有队列的消息会被直接丢弃）。

```java
//第一个参数是交换机，第二个参数是路有密码，第三个是要传递的对象，前两个参数有默认值“”，可不传
amqpTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, Object);
```

#### 自定义Ack机制，默认的自动Ack机制在程序异常时会一直传递消息，直到收到ack

```java
 @Bean
 public SimpleMessageListenerContainer messageContainer() {
     SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory());
     /** 绑定队列，不然不会执行*/
     container.setQueues(queue());
     container.setExposeListenerChannel(true);
     container.setMaxConcurrentConsumers(1);
     container.setConcurrentConsumers(1);
     /** 设置确认模式自动 */
     container.setAcknowledgeMode(AcknowledgeMode.MANUAL);

     container.setMessageListener(new ChannelAwareMessageListener() {
         @Override
         public void onMessage(Message message, Channel channel) throws Exception {
               
             byte[] body = message.getBody(); 
             System.out.print("收到消息："+body);
             /** 确认消息成功接受,不然会服务器会一直给推送*/
             channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);   
         }
     });
     return container;
 }
```
