## RabbitMQ消息的可靠性

消息的可靠性从三个方面考虑：

1.生产者投递消息到RabbitMQ Server 的可靠性保证。

2.RabbitMQ Server中存储的消息可靠性保证。

3.RabbitMQ Server投递消息到消费者的可靠性保证。

### 生产者消息的可靠性保证

可以概括为以下两点：

1. 消息落库，对消息状态打标。
2. 做二次确认，回调检查。

**Conform模式**

生产者通过调用channel.confirmSelect方法将信道设置为confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都会被指派一个唯一的ID（从1开始），一旦消息被投递到所有匹配的队列之后，RabbitMQ就会发送一个确认（Basic.Ack）给生产者（包含消息的唯一deliveryTag和multiple参数），这就使得生产者知晓消息已经正确到达了目的地了。

**实现方式**

串行confirm模式：producer每发送一条消息后，调用waitForConfirms()方法，等待broker端confirm，如果服务器端返回false或者在超时时间内未返回，客户端进行消息重传。

批量confirm模式：producer每发送一批消息后，调用waitForConfirms()方法，等待broker端confirm。

异步confirm模式：提供一个回调方法，broker confirm了一条或者多条消息后producer端会回调这个方法。 我们分别来看看这三种confirm模式（**推荐**）

> confirm机制确保的是消息能够正确的发送至RabbitMQ，这里的“发送至RabbitMQ”的含义是指消息被正确的发往至RabbitMQ的交换器，如果此交换器没有匹配的队列的话，那么消息也将会丢失。

1. 使用mandatory 设置true。

2. 利用备份交换机（alternate-exchange）：实现没有路由到队列的消息。

   ```java
   void basicPublish(String exchange, String routingKey, boolean mandatory, boolean immediate, BasicProperties props, byte[] body)
               throws IOException;
   
   /**
        * 当mandatory标志位设置为true时，如果exchange根据自身类型和消息routeKey无法找到一个符合条件的queue， 那么会调用basic.return方法将消息返回给生产者<br>
        * 当mandatory设置为false时，出现上述情形broker会直接将消息扔掉。
        */
   
       /**
        * 当immediate标志位设置为true时，如果exchange在将消息路由到queue(s)时发现对于的queue上没有消费者， 那么这条消息不会放入队列中。
        当immediate标志位设置为false时,exchange路由的队列没有消费者时，该消息会通过basic.return方法返还给生产者。
        * RabbitMQ 3.0版本开始去掉了对于immediate参数的支持，对此RabbitMQ官方解释是：这个关键字违背了生产者和消费者之间解耦的特性，因为生产者不关心消息是否被消费者消费掉
        */
   ```

   使用mandatory 设置true的时候有个关键点要调整，生产者如何获取到没有被正确路由到合适队列的消息呢？通过调用channel.addReturnListener来添加ReturnListener监听器实现，只要发送的消息，没有路由到具体的队列，ReturnListener就会收到监听消息。

   ```java
   channel.addReturnListener(new ReturnListener() {
               public void handleReturn(int replyCode, String replyText, String exchange, String routingKey, AMQP
                       .BasicProperties basicProperties, byte[] body) throws IOException {
                   String message = new String(body);
                   //进入该方法表示，没路由到具体的队列
                   //监听到消息，可以重新投递或者其它方案来提高消息的可靠性。
                   System.out.println("Basic.Return返回的结果是：" + message);
               }
    });
   ```

   

### 消费端的可靠性保证

关闭autoACK，消费消息后手动ACK和NACK。

### MQ消息的可靠性存储

集群镜像模式。