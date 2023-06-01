# █ JmsTemplate

# 一. 概述

# 二. 建立连接

## 1. java 代码方式

## 2. Spring 配置文件方式



# 三. 发送消息

`JmsTemplate` 提供了三种消息发送方式, 且每种方式针对 `destination` 的形式有三个重载方法, 共计 9 个方法:

## 1. 基本的发送

在 MessageCreator 中创建消息, 然后将消息发送到指定的目的地 Destination,

可以只使用 destinationName , 会自动创建 destination, 默认类型为 queue,

也可以不指定 destination, 但需要提前设置 jmsTemplate 的默认目的地.

```java
public void send(Destination destination, MessageCreator messageCreator);
public void send(String destinationName, MessageCreator messageCreator);
public void send(MessageCreator messageCreator);
```

以上三个方法, 都使用 MessageCreator 来创建消息, 使用方法如下:

```java
jmsTemplate.send("queueDest", new MessageCreator() {
    @Override
    public Message createMessage(Session session) throws JMSException {
        // 在这里创建消息, 并对消息进行操作
        TextMessage msg = session.createTextMessage(message);
        msg.setJMSDeliveryMode(DeliveryMode.PERSISTENT);
        return msg;
    }
});
```

如果想在使用 destinationName 时自动创建 topic 类型的目的地, 可以通过下面的方法进行设置

```java
// 是否默认使用 topic 类型的 destination, 默认为 false, 即默认使用 queue
jmsTemplate.setPubSubDomain(true);
```

## 2. 转换并发送

使用基本发送时, 我们需要在 MessageCreator 中创建具体的消息类型, 如果想直接把数据丢给 JmsTemplate, 由它自动创建消息并发送, 可以采用这几种方法, 其中 destination 的规则与基本发送方法一样. 

```java
public void convertAndSend(Destination destination, Object message);
public void convertAndSend(String destinationName, Object message);
public void convertAndSend(Object message);    
```

使用这种方法, 只需要一行代码即可完成消息发送工作

```java
jmsTemplate.convertAndSend(destinationName, object);
```

## 3. 转换, 后处理再发送

如果需要为消息添加一些 Jmx 属性, 比如为消息进行分组等, 需要在创建消息后, 发送消息前对消息进行操作, 可以采用以下方法

```java
public void convertAndSend(Destination destination, Object message, MessagePostProcessor postProcessor);
public void convertAndSend(String destinationName, Object message, MessagePostProcessor postProcessor);
public void convertAndSend(Object message, MessagePostProcessor postProcessor)
```

这三个方法是 "**转换并发送**" 重载方法, 区别在于增加了一个 **MessagePostProcessor ** 接口, 允许用户在这个接口的实现类中对发送前的消息进行一些处理

```java
String message = "a message for test convertProcessAndSend.";
jmsTemplate.convertAndSend("myDes", message, new MessagePostProcessor() {
    @Override
    public Message postProcessMessage(Message msg) throws JMSException {
        // 在这里对消息进行进一步的处理
        msg.setIntProperty("order", 1);
        return msg;
    }
});
```

# 四. 接收消息

JmsTemplate 提供了四种消息接收方式, 且每种方式针对 destination 的形式有三个重载方法, 共计 12 个方法:

注意 JmsTemplate 所有的显式接收方法，都是阻塞式的——阻塞线程，等待接收，直到超过了预先设定的receiveTimeout，单位是毫秒。而它的初始值是0，表示无限等待。 

```java
// 设置线程阻塞的超时时间, 单位毫秒, 默认0, 无限等待
jt.setReceiveTimeout(3*1000);
```

## 1. 基本接收

从指定的 destination 接收消息,  与消息发送方法类似, 根据 destination 的指定方式有三个重载方法

```java
public Message receive(Destination destination);
public Message receive(String destinationName);
public Message receive();
```

默认情况下返回的对象为 `Message` 类型, 需要在后续的操作中转换为具体类型才能处理.

```java
Message message = jt.receive(DESTINATION_NAME);
// 将消息转换为具体消息类型再处理
if (message != null && message instanceof TextMessage) {
    String text = ( (TextMessage) message ).getText();
    System.out.println(text);
}
```

## 2. 接收并转换

JmsTemplate 提供了直接获取消息中包含对象的方法, 但返回的是 Object 类型, 如有需要仍需进行类型转换

但已能直接获取到消息中的对象, 不再需关注 jms 的五种消息类型

```java
public Object receiveAndConvert(Destination destination);
public Object receiveAndConvert(String destinationName);
public Object receiveAndConvert();
```

 示例如下

```java
Object data = jmsTemplate.receiveAndConvert(DESTINATION_NAME);
if (data != null && data instanceof String) {
    System.out.println(data);
} 
```

## 3. 带有选择器的接收

jms 支持带选择器的消费者, JmsTemplate 也提供了带选择器的接收方法

```java
public Message receiveSelected(Destination destination, String messageSelector);
public Message receiveSelected(String destinationName, String messageSelector);
public Message receiveSelected(String messageSelector);
```

选择器的选择条件, 用 sql 语法给出, 且筛选条件为 message 中设置的 property. 如果需要在接收端筛选消息, 则在发送时就应该为消息设置消息属性, 如下

```java
// 发送端
String message = "a message for test convertProcessAndSend.";
jt.convertAndSend(DESTINATION_NAME, message, new MessagePostProcessor() {
		public Message postProcessMessage(Message message) throws JMSException {
            // 设置消息属性, 以供接收端进行选择
         	message.setIntProperty("order", 1);
            return message;
         }
	});

// 接收端
// string 类型的选择器, 根据消息属性进行选择
String messageSelector = "order = 1";
Message message = jt.receiveSelected(DESTINATION_NAME, messageSelector);
 
if (message != null && message instanceof TextMessage) {
    String text = ((TextMessage) message).getText();
    System.out.println(text);
}
```

## 4. 带有选择器的接收并转换

将选择器和转换器叠加, 就是第四类方法:

```java
public Object receiveSelectedAndConvert(Destination destination, String messageSelector);
public Object receiveSelectedAndConvert(String destinationName, String messageSelector);
public Object receiveSelectedAndConvert(String messageSelector);
```

demo

```java
jt.setReceiveTimeout(3 * 1000); // in milliseconds
String messageSelector = "order = 1";
Object data = jt.receiveSelectedAndConvert(DESTINATION_NAME, messageSelector);
if (data != null && data instanceof String) {
    System.out.println(data);
}
```

## 5. 使用消息监听器

除了使用上述四种同步的消息接收方法, jmsTemplate 还可以使用消息监听器, 在获取到消息时自动处理

要使用消息监听器, 需要实现 `javax.jms.MessageListener` 接口, 创建一个监听器类, 并在启动时向 broker 注册这个监听器

### 1) 最基本的监听器

最简单的监听器, 只接收消息, 不涉及消息的转换等处理, 实现自 `javax.jms.MessageListener`, 

其中 `onMessage` 方法只接收一个参数, 需要在方法中对消息进行转换等处理, 且不能直接发送回信等

```java
public class SpringJMSListener implements MessageListener{
    @Override
    public void onMessage(Message msg) {
        TextMessage textMessage = (TextMessage)msg;
        try{
            System.out.println("listener 收到消息:");
            System.out.println(textMessage.getText());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 2) 带 session 的监听器

spring 提供了一个 `org.springframework.jms.listener.SessionAwareMessageListener` 接口, 可以获取到 session, 

其中 `onMessage` 方法接收两个参数, 可以获取到 session, 通过这个 session 可以发送回信等操作

该接口是泛型接口, 可以直接将消息转换为指定类型

```java
public class SessionListener implements SessionAwareMessageListener<TextMessage> {  
    public void onMessage(TextMessage message, Session session) throws JMSException {
        System.out.println("收到一条消息");
        System.out.println("消息内容是：" + message.getText());
        Destination replyTo = message.getJMSDestination();
        MessageProducer producer = session.createProducer(replyTo);
        Message textMessage = session.createTextMessage("ConsumerSessionAwareMessageListener。。。");
        producer.send(textMessage);
    }
}  
```

### 3) 消息监听适配器 

`MessageListenerAdapter` 类实现了 `MessageListener` 接口和 `SessionAwareMessageListener` 接口，它将接收到的消息进行类型转换，然后再交给一个具体的目标处理器进行处理。

MessageListenerAdapter会把接收到的消息做如下转换：

- TextMessage转换为String对象；
- BytesMessage转换为byte数组；
- MapMessage转换为Map对象；
- ObjectMessage转换为对应的Serializable对象。

#### i. 指定目标处理器

由于 MessageListenerAdapter 要将转换后的消息交给一个目标处理器, 那么我们在定义一个 Adapter 的时候就需要为它指定这样一个目标类。

- 这个目标类我们可以通过 MessageListenerAdapter 的构造方法参数指定，如：

  ```xml
  <bean id="messageListenerAdapter" class="org.springframework.jms.listener.adapter.MessageListenerAdapter"> 
      <constructor-arg>  
          <bean class="com.tiantian.springintejms.listener.ConsumerListener"/>  
      </constructor-arg>  
  </bean>  
  ```

-  也可以通过它的delegate属性来指定，如:

  ```xml
  <bean id="messageListenerAdapter" class="org.springframework.jms.listener.adapter.MessageListenerAdapter"> 
      <property name="delegate">  
          <bean class="com.tiantian.springintejms.listener.ConsumerListener"/>  
      </property>  
  </bean>  
  ```

#### ii. 指定处理方法

如果真正的目标处理器是一个 `MessageListener` 或者是一个 `SessionAwareMessageListener` ，那么 Spring 将直接使用接收到的Message 对象作为参数调用它们的 onMessage 方法，而不会再利用反射去进行调用.

如果指定的目标处理器是一个普通的 Java 类时, Spring 将转换之后的对象作为参数, 通过反射去调用真正的目标处理器的处理方法，具体的方法要 `MessageListenerAdapter` 的 `defaultListenerMethod` 属性来决定的，没有指定该属性时，Spring 会默认调用目标处理器的 `handleMessage` 方法。 

```xml
<bean id="messageListenerAdapter" class="org.springframework.jms.listener.adapter.MessageListenerAdapter"> 
    <property name="delegate">  
        <bean class="com.tiantian.springintejms.listener.ConsumerListener"/>  
    </property>  
    <!-- 指定处理方法 -->
    <property name="defaultListenerMethod" value="receiveMessage"/>  
</bean>  
```

#### iii. 指定自动回信目的地

当我们用于处理接收到的消息的方法的返回值不为空的时候，Spring 会自动将它封装为一个JMS Message，然后自动进行回复。 

如果该消息的发送发设置了`setJMSReplyTo ` 的目的地, `MessageListenerAdapter` 会将消息发往该目的地;

如果消息没有设置 `JMSReplyTo ` 目的地, 可以在 `MessageListenerAdapter` 设置了默认回信目的地

```xml
<bean id="messageListenerAdapter" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">    
    <property name="delegate">  
        <bean class="com.tiantian.springintejms.listener.ConsumerListener"/>  
    </property>  
    <property name="defaultListenerMethod" value="receiveMessage"/>  
    <!-- 指定默认回信目的地 -->
    <property name="defaultResponseDestination" ref="defaultResponseQueue"/>  
</bean>  
```

#### vi. 使用 @JmsListener 注解

先在 `spring-jms.xml` 中添加命名空间

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:jms="http://www.springframework.org/schema/jms"
       xsi:schemaLocation="
                           http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util
                           http://www.springframework.org/schema/util/spring-util.xsd
                           http://www.springframework.org/schema/jms
                           http://www.springframework.org/schema/jms/spring-jms.xsd">
```

添加 jms 注解扫描, 并配置`containFactory`

```xml
<!-- 启动 jms 注解扫描 -->
<jms:annotation-driven/>
<!-- listenerContainer 工厂, 配置后就可以使用 @JmsListener -->
<bean id="jmsListenerContainerFactory" class="org.springframework.jms.config.DefaultJmsListenerContainerFactory">
    <property name="connectionFactory" ref="jmsFactory"/>
    <property name="sessionTransacted" value="true"/>
    <property name="concurrency" value="3-10"/>
</bean>
```



### 4) 注册监听器

通过以上三种方式定义的 listener, 还需要使用 springJms 的 `ListenerContainer` 来注册

如果需要为监听器配置选择器, 可以为 container 添加一个 `messageSelector` 属性, value 值就是选择条件

```xml
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="messageSelector" value="order=1"/>
    <property name="connectionFactory" ref="jmsFactory"/>
    <property name="destination" ref="destinationTopic" />
    <property name="messageListener" ref="messageListener"/>
</bean>
```

# 五. 事务控制

如果监听器方法中顺利执行, 则消息签收成功.

如果监听器方法中抛出异常, 则消息签收失败, broker 将消息重发, 如果重发次数超过限制, 消息就会被发往死信队列





Local resource transactions can simply be activated through the `sessionTransacted` flag on the listener container definition. 

本地资源事务, 能很容易的通过监听器容器中的`sessionTransacted`  来激活, 

Each message listener invocation will then operate within an active JMS transaction, 

监听器的每次调用, 都会在一个激活的 jms 事务中进行, 

with message reception rolled back in case of listener execution failure. 

消息处理失败后, 消息的接收将会被回滚



Sending a response message (via `SessionAwareMessageListener`) will be part of the same local transaction, 

通过`SessionAwareMessageListener` 进行的回信操作, 也会成为同个jms 事务中的一部分

but any other resource operations (such as database access) will operate independently. 

但对于其他资源的操作, 比如数据库访问, 则将会独立进行

This usually requires duplicate message detection in the listener implementation, 

这通常需要在监听器实现中定义多个消息目的地

covering the case where database processing has committed but message processing failed to commit. 

以包括数据库操作成功但消息发送失败





问题: 

1. 是否会自动签收消息, 无论消费者是否顺利执行, 有没有抛出异常
2. 如果数据库抛出异常, 消息的接收是否会跟数据库事务一起回滚
3. 如果在消息处理过程中向另一个队列发送了消息, 数据库事务回滚时, 消息有没有被发出去
4. 需要使用 JTA 事务控制?