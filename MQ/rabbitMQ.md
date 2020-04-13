## mq 消息中间件

#### 一、过程理解

> 生产者  ----connetion(channel)---->   rabbitmqServer【  Broker(exchange  ------->  queue  ) 】 <-----connection(channel)-------     消费者

#### 二、概念理解

connection：指的是生产者或消费者与 rabbitMQ 服务器的tcp连接。

Channel ：指的是基于 connection连接的客户端与服务器的会话（虚拟连接），也叫通道。目的是为了隔离。即每次会话与其它会话之间是不受干扰的。

Exchange ：交换机，它的作用是将生产者设置的 routingKey 来匹配对应的队列（消费者监听的队列）。**这里需要注意的是，当 Exchange 找不到routingKey对应的队列时，消息将会被丢弃。另。生产者发送消息时不指定 exchange 时，即表示使用默认的 exchange。此时routingKey 的设置将要与 消费者监听的队列同名。（不然会出现找不到对应的消费队列，后使消息被抛弃）**

Exchange在定义时，要指名类型：**fanout**、**direct**、**topic**、**headers**

Fanout：对应的是 rabbitmq工作模式中的 publish/subscribe（发布与订阅）

Direct:    对应的是 rabbitmq 工作模式中的 route（路由）

Topic：对应的是 rabbitmq 工作模式中的 topics （主题匹配）

Headers：对应的就是工作模式中的 headers




#### 三、rabbitMQ  的工作模式

**simple,work** ，简单模式，不涉及到 exchange 的设置。simple，指的是一个生产者，一个消费者。work 相对 simple 而言，是将一个消费者，变成多个消费者。work指的是**一个队列上，监听了多个消费者【不同的 channel，同一个 queue】**（因为多个消费者，采用轮询的方式来处理消费。多个消费者最终只有一个消费者能拿到消息消费）

**Publish/subscribe**,（exchange 的 type 要设置为fanout）一个消息可以被不同消费者队列进行消费， 指的是一个消息过来。通过 exchange对 routingKey的匹配，来找到对应的队列（可能是多个队列），最终队列的 监听消费者，进行消费。

**Route**,(exchange 的 type 要设置为direct) 也是一个消息可以被不同消费者队列进行消费。消费者队列绑定交换机时，指定了 routingKey。而生产者消息在 pushlish 的时候，指定了 routingKey，通过 exchange 进行匹配，将消息发送到指定的队列中。需要注意的是**消费者在绑定队列到 exchange 的时候，可以指定多个 routingKey**   相对 pub/sub 模式而言。多了一层 routingKey 的判断。只有和生产者发送时设置的 routingKey相等的消费队列，才可以被消费。当然，如果不同队列设置了相同的 routingKey时，其实就是 pub/sub (fanout)的模式.也就是为什么说，pub/sub 模式时。routingKey 设置为""的原因。

   区别：

> pub/sub【fanout】，只要消费者队列绑定的是同一个交换机。那么消息来时，所有消费者队列都收到消息。
>
> route【direct】,虽然多个消费者队列绑定的是同一个交换机。但因为设置了不同的 routingKey。所以，消息来时，与之对应的消费者队列能收到消息。**route 具备 pub/sub 相应的功能**。

  **Topic**，算是 route 模式的改进。route 模式的匹配规则是相等。即生产者设置队列的 routingKey 与消费者队列设置routingKey，是相等的。而 topic，支持模糊匹配。通过对 routingKey 加上*，# .

> *只能匹配一个词。如msg_\*  ,只能匹配 msg_sms,msg_email 这类型的 routingKey
>
> \#支持一个或多个词的匹配。如 msg_\#  ,可以匹配到 msg_sms, msg_sms_abc



**headers**  模式。这种模式绑定交换机的时候，不再设置 routingKey了。而是设置一个 key/value 对，即生产者在发送的时候，不再设置 routingKey（routingKey=”“），而是设置了 k-v对（可以是多对）。



#### 四、对应的方法

> connection(host,port,user,pwd)  不管是生产者，还是消费者，首先要进行与 rabbitmq 服务器的 tcp 连接

>  channel=channelDeclare()    定义通道。不对是生产者，还是消费者。都是基于通道进行与服务器的操作。

> Channel.QueueDeclare(queueName)  定义队列。不管是产者还是消费者，都建议定义队列。因为，如果先启动生产者，队列还没有定义。消息不能发出。如果先启动消费者。队列不存在。也会报错。

>Channel.ExchangeDeclare(名称,类型)  定义交换机。同样生产者或消费者中都需要定义。

> Channel.queueyBind(队列名称，交换机名称，routingKey)，将队列绑定到交换机。发布订阅模式中，routingKey 为空。

> Channel.basicPushlish(交换机名,routingKey,附加参数,消息内容)  生产者 生产一个消息，这里需要注意的是：**simple、work 模式下交换机名可以设置为"",而 routingKey 通常设置为队列名**,而  ,pub/sub 模式(fanout)下，routingKey设置为""。

> Channel.basicConsume(是否自动回复ack,消费方法) ，消费者进行消费。

> Channel.close()  道通关闭。一定要先进行 channel 关闭，再进行 connnetion 的关闭。而消费者一般不手动执行关闭，因为要持续监听消息的到来。

