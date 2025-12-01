---
{"dg-publish":true,"permalink":"/kafka/kafka-delay-queue/"}
---


## 什么是延迟队列？
延迟队列是一种特殊的消息队列，可以将消息或任务推迟到指定的时间再进行处理。它通常用于处理需要在未来某个时间点执行的任务，如定时任务、延迟通知等。延迟队列允许开发人员根据任务的延迟要求进行灵活的调度和处理。

## 如何使用Kafka实现延时队列？
Kafka本身并没有提供原生的延时队列功能，但我们可以通过一些技术手段实现演示队列的功能。

#### Thread.sleep()
结论：不靠谱的实现方式
原因：在轮询kafka拉取消息的时候，kafka会返回由max.poll.records配置指定的一批消息，当程序不能在max.poll.interval.ms配置的期望时间内处理这些消息的话，kafka就会认为这个消费者已经挂了，会进行rebalance，同时你这个消费者就无法再拉取到任何消息了

#### 时间轮
[时间轮实现延时队列](https://www.jianshu.com/p/b0894e0c33de)


#### 时间戳
Kafka的延迟消息实现原理比较简单，主要涉及到消息的key和时间戳。在消息的key中，可以设置一个时间戳，表示消息的延迟时间。在生产者发送消息时，将消息发送到"delayed-messages" Topic中，并设置消息的key中的时间戳。消费者进程会定期从"delayed-messages" Topic中消费消息，检查消息的key中的时间戳是否已经过期。如果时间戳已经过期，则将消息重新发送到目标Topic中，例如"target-messages"。如果时间戳还未过期，则将消息重新发送到"delayed-messages" Topic中，并设置一个新的延迟时间戳。这样，就可以实现延迟队列的功能。

![](https://pic.imgdb.cn/item/65325102c458853aefac04f0.webp)

##### 实现步骤
1.创建一个专门的Topic用于存储延迟消息，例如"delayed-messages"。可以使用Kafka命令行工具或Kafka API进行创建。

2.在消息的key中设置延迟时间戳。可以使用当前时间戳加上延迟时间作为key，例如："key":"message_body"。可以使用Kafka API发送消息到"delayed-messages" Topic中。

3.启动一个消费者进程，用于消费"delayed-messages" Topic中的消息。可以使用Kafka API实现消费者进程。

4.在消费者进程中，检查消息的key中的时间戳是否已经过期。可以使用当前时间戳与消息的key中的时间戳进行比较。如果时间戳已经过期，则将消息重新发送到目标Topic中，例如"target-messages"。可以使用Kafka API实现消息的重新发送。

5.如果时间戳还未过期，则将消息重新发送到"delayed-messages" Topic中，并设置一个新的延迟时间戳。可以使用Kafka API实现消息的重新发送，并在消息的key中设置新的延迟时间戳。

6.等待一定时间后，重复执行第4和第5步，直到消息的key中的时间戳已经过期。

通过以上步骤，就可以实现Kafka中的延迟队列功能。需要注意的是，消费者进程需要定期从"delayed-messages" Topic中消费消息，并检查消息的key中的时间戳是否已经过期。可以根据具体的应用场景设置不同的延迟时间。