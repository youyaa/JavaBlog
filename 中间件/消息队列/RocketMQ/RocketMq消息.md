## RocketMQ发送消息

1. **发送同步消息**

   ```java
   public class SyncProducer {
   	public static void main(String[] args) throws Exception {
       	// 实例化消息生产者Producer
           DefaultMQProducer producer = new 	           DefaultMQProducer("please_rename_unique_group_name");
       	// 设置NameServer的地址
       	producer.setNamesrvAddr("localhost:9876");
       	// 启动Producer实例
           producer.start();
       	for (int i = 0; i < 100; i++) {
       	    // 创建消息，并指定Topic，Tag和消息体
       	    Message msg = new Message("TopicTest","TagA" ,
           	("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
           	  // 发送消息到一个Broker,并等待发送结果返回
               SendResult sendResult = producer.send(msg);
               // 通过sendResult返回消息是否成功送达
               System.out.printf("%s%n", sendResult);
       	}
       	// 如果不再发送消息，关闭Producer实例。
       	producer.shutdown();
       }
   }
   ```

2. **发送异步消息**

   发送消息耗时较长，但发送者不想等待。

   ```java
   producer.send(msg, new SendCallback() {
         public void onSuccess(SendResult sendResult) {
          }
         public void onException(Throwable throwable) {
         }
   ```

3. **发送单向消息**

   发送不重要的消息，不关心是否发送成功，例如：发送日志。

   ```java
   // 发送单向消息，没有任何返回结果
     producer.sendOneway(msg);
   ```

## RocketMQ消费消息

Rocket支持两种消费方式：

1. **集群消费**：集群中的机器共享消息。
2. 广播消费**：集群中的机器都能收到完整的消息。

发送消息时不做区分，在订阅消息时，可通过如下方式设置消费方式：

```java
consumer.setMessageModel(MessageModel.CLUSTERING);   //集群消费
consumer.setMessageModel(MessageModel.BROADCASTING); //广播消费
```



## 消息的顺序性

RocketMQ是一个先进先出(FIFO)的队列，支持严格的顺序。

### 顺序发送

保证消息的局部顺序，即通过将需要被顺序消费的消息发送到同一个队列中即可。

```java
SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
               @Override
               public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                   Long id = (Long) arg;  //根据订单id选择发送queue
                   long index = id % mqs.size();
                   return mqs.get((int) index);
               }
           }, orderList.get(i).getOrderId());//订单id
```

可以同如上的方式，自定义消息的队列，通过消息的唯一标示和队列的个数取模，保证同一个标识下的消息发送到同一个队列中。

### 顺序消费

```java
consumer.registerMessageListener(new MessageListenerConcurrently() {
                 public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext Context) {
               
                 }
             }
        );
```

消费者在注册监听器时，传入MessageListenerConcurrently即可，对于同一个Queue，会有同一个消费线程来消费。

## 延迟消息

发送消息时设置延迟等级：

```java
// 设置延时等级3,这个消息将在10s之后发送(现在只支持固定的几个时间,详看delayTimeLevel)
 message.setDelayTimeLevel(3);
```

RocketMQ不支持任意的延迟，现支持如下的等级：

```java
// org/apache/rocketmq/store/config/MessageStoreConfig.java
private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";
```

## 过滤消息

支持两种过滤方式：

1. tag过滤。
2. SQL语法过滤。

## 事务消息

