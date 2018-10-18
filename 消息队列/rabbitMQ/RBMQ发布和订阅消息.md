RBMQ发布和订阅消息
===

## exchange
参考翻译自: [RabbitMQ官网](https://www.rabbitmq.com/tutorials/tutorial-three-java.html)
*************
生产者并非将消息直接发送到queue,而是发送到`exchange`中,具体将消息发送到特定的队列还是多个队列,或者是丢弃,取决于`exchange`的类型
### exchange的类型
* direct
* topic
* headers
* fanout
## bindings(绑定队列)
当生产/消费`exchange`时,可以绑定队列到`exchange`中:
```
channel.queueBind(queueName, EXCHANGE_NAME, "");
```
bind方法将一个队列和一个exchange绑定到一起,第三个参数 routing key,相当于描述这个绑定关系的key,会根据不同类型的exchange,来决定将消息发送到queue中,多个queue中,或者消费时,将消息递送到绑定的queue中
### fanout类型
`fanout`将收到的所有消息发送到所有的已知队列中.
#### 定义一个fanout类型的exchange
```
channel.exchangeDeclare("logs", "fanout");
```
#### 发送消息到exchange中
```
channel.basicPublish( "logs", "", null, message.getBytes());
```
#### 消费exchange

当我们消费一个fanout类型的exchange时,exchange发送消息到多个queue,而我们想要消费所有的queue,而不是其中一部分,而我们不知道消息具体被发送到哪个queue,也没法指定多个queue,此时需要一个临时queue
##### Temporary Queue
```
String queueName = channel.queueDeclare().getQueue();
```
这样定义了一个临时queue,这个队列非持久,消费者独有的,会被自动删除,当消费者断开连接时,临时queue自动被删除

为了告诉exchange发送消息到我们定义的临时queue,需要进行绑定操作:
```
channel.queueBind(queueName, "logs", "");
```
### direct类型
#### 生产队列
```
channel.queueBind(queueName, EXCHANGE_NAME, "black");
channel.queueBind(queueName1, EXCHANGE_NAME, "white");

```
当exchange为`direct`类型时,会将exchange和`queuename`绑定到一起绑定key为`black`,将exchange和`queuename1`绑定到一起,绑定key为`white`.`fonout`类型会忽略第三个参数,当发送消息时:
```
channel.basicPublish(EXCHANGE_NAME, "black", null, message.getBytes());
channel.basicPublish(EXCHANGE_NAME, "white", null, message1.getBytes());
```
此时,会将`message`发送到绑定的`queuename`队列中,`message1`发送到绑定的`queuename1`队列中.
> 绑定多个队列到一个binding key时,相当于广播,会将消息发送到多个队列中

#### 消费队列
```
String queueName = channel.queueDeclare().getQueue();

for(String severity : argv){
  channel.queueBind(queueName, EXCHANGE_NAME, severity);
}
```
当消费一个`direct`类型的exchange时,通过binding key 决定消费哪个队列




### `topic`类型
有时候,我们需要发布/订阅消息有多个筛选类型,比如发布一个动物消息:移动速度,颜色,类别.当需要订阅所有同一个颜色的动物时,比如`全部移动速度,黑色,所有类别.此时涉及到模糊匹配,使用`fanout`和`direct`类型不符合要求,此时要用到`topic`类型
#### 发布消息
```
 channel.basicPublish(EXCHANGE_NAME, "quick.orange.rabbit", null, message.getBytes());
 channel.basicPublish(EXCHANGE_NAME, "lazy.orange.elephant", null, message.getBytes());
```
发布了两个routing key,一个`快,橙色,兔子`,一个`慢,橙色,大象`,这两个routing key的消息发到了不同的队列中
#### 消费消息
```
channel.queueBind(queueName, EXCHANGE_NAME, "*.orange.*");
```
通过绑定队列,到模糊匹配routing key上,当所有符合匹配规则的routing key的消息都会被消费者队列所消费
### 关于`exchange`的命令
#### 列出exchanges
```
sudo rabbitmqctl list_exchanges
```
列出服务器上的exchanges,`amq.*`开头的exchange和default(没有名字)的exchange是默认创建的.一般不大可能直接使用.
> 未命名的exchange  
未命名的exchange默认将消息分发给routingKey定义的名称的队列