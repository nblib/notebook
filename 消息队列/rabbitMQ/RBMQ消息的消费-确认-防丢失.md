RBMQ消息的消费,确认,防丢失
===
参考翻译自: [RabbitMQ官网](https://www.rabbitmq.com/tutorials/tutorial-three-java.html)
*************
## 向多消费者分发消息

当有多个消费者时:
1. 使用Round-robin方法,也就是循环发送,一个消息发送给一个消费者,下一条消息发送给下一个消费者,依次循环.
2. 每个消息只发送给一个消费者
## 消息确认机制
1. 每个消费者消费后,需要反馈给MQ,确认消息已经被消费,MQ可以删除掉
2. 如果一个消费者挂掉(channel关闭,connection 关闭,或者TCP连接丢失),MQ便会得知消费者挂掉,如果此时还有别的消费者,MQ将消息发给存活的消费者
3. 消费者消费消息没有超时的概念,消费者处理一条消息不管都长时间只要没有挂掉,MQ都不会重发消息
4. 确认消息必须和接收消息是同一个channel,如果使用不同的channel确认消息,会引发channel-level protocol exception

> 如果忘记确认消息,MQ将会吃掉很大的内存来维持未确认的消息,而且一旦一个消费者挂掉,那么它没有确认的消息会再次发给其他消费者.    

> 可以通过使用`rabbitmqctl`打印为确认的消息: 
```
sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged
```
## 消息防丢
当MQ宕机时,内存中的消息队列和消息会丢失,为了避免可以通过设置避免队列和消息丢失
### 避免队列丢失
声明队列时,设置`durable`为`true`
```
boolean durable = true;
channel.queueDeclare("hello", durable, false, false, null);
```
> 一个队列只有在第一次声明的时候,设置的参数才起作用,当对一个已经存在的消息队列再次声明,设置参数的时候,不会起作用.

### 避免消息丢失
通过设置`MessageProperties`的值为`PERSISTENT_TEXT_PLAIN`将消息持久化:
```
channel.basicPublish("", "task_queue",
            MessageProperties.PERSISTENT_TEXT_PLAIN,
            message.getBytes());
```
> 消息持久化到磁盘,并不能完全避免消息丢失,因为在写到磁盘时也有缓存.
## 均匀分发消息(Fair dispatch)
当MQ中消息有的容易处理,有的处理慢,会导致个别的消费者的压力大,比如第一个,第三个等消息处理起来慢,如果有两个消费者,这样会导致第一个消费者压力大,为了避免这样情况,可以通过设置参数,只有消费者消费完一个消息才会给它再分发消息:
```
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```
> 队列大小  
如果所有消费者都慢,会导致队列消息满了,此时需要增加消费者或者使用其他策略