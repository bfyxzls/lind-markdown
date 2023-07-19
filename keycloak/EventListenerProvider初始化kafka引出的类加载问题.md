# EventListenerProvider初始
keycloak提供的事件处理机制，可以通过实现EventListenerProvider接口来实现自定义的事件处理逻辑。在keycloak启动时，会通过ServiceLoader机制加载所有的EventListenerProvider实现类，并将其注册到keycloak的事件处理机制中。
* 构造方法，在每个keycloak后台操作时，它都会重新构建实例
* OnEvent方法，在事件发生时执行，不会出现类加载问题，因为这样类已经被加载了

# EventListenerProviderFactory
EventListenerProviderFactory是进行事件处理器的生产工厂，用于创建EventListenerProvider实例。在keycloak启动时，会通过ServiceLoader机制加载所有的EventListenerProviderFactory实现类，并将其注册到keycloak的事件处理机制中。
* init方法：keycloak启动时会执行，用于初始化EventListenerProviderFactory实例，可以在此方法中进行一些初始化操作。
* postInit方法：keycloak启动时会执行，在init方法之后，会执行这个方法
* create方法：在kc后台开启这个EventListenerProviderFactory之后，每次请求都会执行这个create方法，对于它生产的provider对象，可能考虑使用单例的方式， 避免每次请求都创建一个新的对象

# 问题
* 问题描述：在EventListenerProviderFactory的init方法中，通过kafka发送消息，会出现类加载问题，因为在keycloak启动时，kafka的类的加载器还没有被加载，所以会出现类加载问题。
* 解决：需要将类加载器这块，修改成当前类加载器去加载对应的文件，如下代码解决了类无法加载的问题
```java
  public KcKafkaConsumer(Properties properties, String topic) throws ClassNotFoundException {
    // 需要使用当前类加载器，否则会出现无法加载StringDeserializer的情况
    Class<?> stringDeserializerClass =
        getClass().getClassLoader().loadClass("org.apache.kafka.common.serialization.StringDeserializer");
    properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, stringDeserializerClass);
    properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, stringDeserializerClass);

    this.kafkaConsumer = new KafkaConsumer<>(properties);
    this.kafkaConsumer.subscribe(Collections.singleton(topic));
  }
```