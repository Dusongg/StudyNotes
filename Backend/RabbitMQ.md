- 为什么要使用消息队列

![image-20240903160417835](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240903160417835.png)

# 1 认识

## 1.1 MQ分类

- 有Broker
  - 重Topic —— 在整个broker中，依据topic来进行消息中转。在重topic的MQ中必然需要topic —— kafka
  - 轻Topic —— topic只是一种中转模式 —— rabbitMQ

- 无Broker

## 1.2 安装

```bash
# latest RabbitMQ 3.13
docker run \
	-e RABBITMQ_DEFAULT_USER=dusong \  #默认账号和密码均为：guest
	-e RABBITMQ_DEFAULT_PASS=123123 \
	-d \  #detached mode
	-v mq-plugins:/plugins \   #插件挂载
	--rm \
   	--name rabbitmq \
    -p 5672:5672 \    #消息通信端口
    -p 15672:15672 \  #管理界面端口
    rabbitmq:3.13-management
```

## 1.3 基本流程

![image-20240904110326213](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240904110326213.png)

- exchange只能转发消息，不能存储消息
- 通过bind将queue绑定到exchange



# 2 [Work模型](https://www.rabbitmq.com/tutorials/tutorial-two-go#preparation)

- 多个消费者绑定到一个队列

- 同一个消息只会被一个消费者处理

- 通过设置prefetch来控制消费者预取的消息数量（不设置默认平均平均分配）

  ![image-20240904144344585](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240904144344585.png)

  ```go
  err = ch.Qos(
    1,     // prefetch count
    0,     // prefetch size
    false, // global
  )
  ```

  

# 3 交换机

## 3.1 fanout

fanout类型的交换机会**将消息转发给所有绑定到改交换机的队列**

## 3.2 direct

![image-20240904151234823](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240904151234823.png)

```go
err = ch.ExchangeDeclare(
  "logs_direct", // name
  "direct",      // type
  true,          // durable
  false,         // auto-deleted
  false,         // internal
  false,         // no-wait
  nil,           // arguments
)
failOnError(err, "Failed to declare an exchange")

ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

body := bodyFrom(os.Args)
err = ch.PublishWithContext(ctx,
  "logs_direct",         // exchange
  "log", // routing key
  false, // mandatory
  false, // immediate
  amqp.Publishing{
    ContentType: "text/plain",
    Body:        []byte(body),
})
```



## 3.3 [topic](https://www.rabbitmq.com/tutorials/tutorial-five-go)

![image-20240904151944421](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240904151944421.png)





# 4 Golang创建交换机/队列/Publish/Consume/Bind

- 创建交换机

  ```go
  err = ch.ExchangeDeclare(
          "logs_direct", // name
          "direct",      // type
          true,          // durable
          false,         // auto-deleted
          false,         // internal
          false,         // no-wait
          nil,           // arguments
  )
  ```

- 创建队列

  ```go
  q, err := ch.QueueDeclare(
      "hello", // name
      false,   // durable（是否持久化）
      false,   // delete when unused
      false,   // exclusive
      false,   // no-wait
      nil,     // arguments
  )
  ```

- 绑定

  ```go
  err = ch.QueueBind(
          q.Name,        // queue name
          "log",             // routing key
          "logs_direct", // exchange
          false,
          nil
  )
  ```

- 发送

  ```go
  body := "this is log"
  err = ch.PublishWithContext(ctx,
          "logs_direct",         // exchange
          "log", // routing key
          false,                 // mandatory
          false,                 // immediate
          amqp.Publishing{
                  ContentType: "text/plain",
                  Body:        []byte(body),
          })
  ```

- 接收

  ```go
  msgs, err := ch.Consume(
          q.Name, // queue
          "",     // consumer
          true,   // auto ack
          false,  // exclusive
          false,  // no local
          false,  // no wait
          nil,    // args
  )
  ```

  



# 5 可靠性

## 5.1 生产者可靠性

- 生产者重连

- 生产者确认（ack）


## 5.2 MQ可靠性

- 交换机/队列持久化
- 消息持久化

### 5.2.1 Lazy Queue

![image-20240904172117264](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240904172117264.png)

![image-20240904163818387](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240904163818387.png)



## 5.3 消费者可靠性

- 消费者确认机制

  ![image-20240904172521990](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240904172521990.png)

## 5.4 业务幂等性

- 消费者因为保证可靠性可能消费业务多次，因此需要保证业务幂等性

1. 给消息加上uuid
2. 在业务逻辑上做修改 

## 5.4 Golang实现可靠性

> 在使用 RabbitMQ 的 Go 应用程序中，要确保消息的可靠性，通常需要从以下几个方面入手：
>
> ### 1. 确保消息生产者的可靠性
> - **消息确认（Publisher Confirms）：** 开启 RabbitMQ 的发布确认模式。通过调用 `Channel.Confirm()` 方法，让 RabbitMQ 服务器在成功接收并持久化消息后向生产者发送确认。这样可以确保生产者知道消息已被可靠接收。
>   
>   ```go
>   ch.Confirm(false) // 启用发布确认模式
>   confirm := ch.NotifyPublish(make(chan amqp.Confirmation, 1))
>   
>   // 发布消息
>   err = ch.Publish(exchange, routingKey, mandatory, immediate, msg)
>   if err != nil {
>       // 处理发布失败的情况
>   }
>   
>   select {
>   case confirmed := <-confirm:
>       if confirmed.Ack {
>           fmt.Println("消息已确认")
>       } else {
>           fmt.Println("消息未确认")
>       }
>   case <-time.After(time.Second * 5):
>       fmt.Println("消息确认超时")
>   }
>   ```
>
> - **消息持久化（Message Durability）：** 将消息标记为持久化，以确保即使 RabbitMQ 服务器重启，消息也不会丢失。通过设置 `DeliveryMode` 为 `amqp.Persistent` 来实现：
>
>   ```go
>   msg := amqp.Publishing{
>       DeliveryMode: amqp.Persistent,
>       ContentType:  "text/plain",
>       Body:         []byte("Hello, RabbitMQ!"),
>   }
>   ```
>
> ### 2. 确保消息队列的可靠性
> - **队列持久化（Queue Durability）：** 创建队列时，将其声明为持久化队列。这样即使 RabbitMQ 服务器重启，队列依然存在。
>
>   ```go
>   _, err = ch.QueueDeclare(
>       "my_queue",  // 队列名
>       true,        // 是否持久化
>       false,       // 是否自动删除
>       false,       // 是否排他
>       false,       // 是否阻塞
>       nil,         // 其他参数
>   )
>   if err != nil {
>       log.Fatalf("Failed to declare a queue: %s", err)
>   }
>   ```
>
> ### 3. 确保消息消费者的可靠性
> - **手动确认（Manual Acknowledgment）：** 消费者手动确认接收到的消息。这样只有在消息成功处理后，RabbitMQ 才会将其从队列中移除。如果消费者没有确认消息且发生故障，RabbitMQ 会将消息重新投递。
>
>   ```go
>   msgs, err := ch.Consume(
>       "my_queue", // 队列名
>       "",         // 消费者标识
>       false,      // 自动确认
>       false,      // 是否排他
>       false,      // 是否阻塞
>       false,      // 是否在同一个连接上消费
>       nil,        // 其他参数
>   )
>   if err != nil {
>       log.Fatalf("Failed to register a consumer: %s", err)
>   }
>
>   for d := range msgs {
>       // 处理消息
>       fmt.Printf("Received a message: %s", d.Body)
>
>       // 手动确认
>       d.Ack(false)
>   }
>   ```
>
> - **QoS（Quality of Service）：** 设置消费者的 QoS 参数，例如 `prefetch_count`，确保消费者不会一次处理太多消息，从而导致过载。
>
>   ```go
>   err = ch.Qos(
>       1,    // 每次处理一条消息
>       0,    // 消息大小限制（不限制）
>       false, // 是否应用于整个通道
>   )
>   if err != nil {
>       log.Fatalf("Failed to set QoS: %s", err)
>   }
>   ```
>
> ### 4. 容错处理
> - **重试机制：** 在生产者和消费者中实现重试机制，例如使用带有指数回退的重试逻辑，以应对 RabbitMQ 不可用或网络波动的情况。
>
> - **死信队列（DLX）：** 配置死信队列，将处理失败的消息路由到指定的死信队列，方便后续分析和处理。
>
> 通过这些措施，可以有效提高使用 RabbitMQ 时的消息可靠性。



# 6 延迟消息

## 6.1 死信交换机

![image-20240905145924200](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240905145924200.png)

## 6.2 延迟消息插件

### 6.2.1 安装

1. https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/v3.13.0/rabbitmq_delayed_message_exchange-3.13.0.ez

2. 将插件放在该目录

   ![image-20240905153455222](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240905153455222.png)

3. `docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq-delayed-message-exchange`

### 6.2.2 使用

```go
 // 3. 声明延迟交换机
    err = ch.ExchangeDeclare(
        "delay_exchange",               // 交换机名称
        "x-delayed-message",            // 交换机类型
        true,                           // 是否持久化
        false,                          // 是否自动删除
        false,                          // 是否内部使用
        false,                          // 是否等待
        amqp.Table{"x-delayed-type": "direct"}, // 交换机类型的设置
    )
    failOnError(err, "Failed to declare an exchange")

    // 4. 发送消息
    body := "Hello World with delay"
    err = ch.Publish(
        "delay_exchange", // 交换机名称
        "routing_key",    // 路由键
        false,            // 是否强制发送
        false,            // 是否立即发送
        amqp.Publishing{
            ContentType: "text/plain",
            Body:        []byte(body),
            Headers: amqp.Table{
                "x-delay": int32(5000), // 延迟时间，单位为毫秒 (5秒延迟)
            },
        })
```

### 6.2.3 应用场景

- 消息内部维护一个计时器，延迟消息对CPU的消耗较高，适用于延迟时间较短的场景

![image-20240905155732697](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240905155732697.png)
