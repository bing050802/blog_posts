---
layout: post
comments: true
title: jms-如何保证消息的顺序
date: 2017-01-23 09:27:32
tags:
categories:
- MQ
---

### 背景

在特定的业务场景中我们可能需要保证消息消费的顺序性，因此需要消息中间件能提供消息顺序性的保证。JMS规范中有两种消息模型，`queue`和`topic`。 队列先天能保证消息的顺序，这是由于它本身的特性来决定的。但`JMS`不能保证多个队列中消息消费的顺序性，例如先进入队列A的消息并不能保证一定在后进入队列B的消息先被消费。

<!-- more -->

现有的系统架构都是分布式的。有多个消息的发送者和多个消息的消费者。例如订单创建消息和订单支付消息，我们需要保证先消费订单创建消息，然后消费订单支付消息。如果这两种类型的消息使用同一个队列，这绝大多数情况下订单创建消息是先于订单支付消息到达队列的。但是在消费的情况下很有可能是支付消息先到达消费者而订单创建消息后到达消费者（两个消费者是不同的应用，或者同一个应用。就算是同一个，网络也不能保证先发送的消息一定是被先消费）。

### 那如何保证消息的顺序性呢

#### 乱序发送-顺序消费

也就是说消息在发送，传输和接收过程中可能是乱序的，但消费者在接收到消息之后并不立即处理，而是先将消息排序然后再处理。我们可以在消息中添加先要被消费的消息ID（类似父ID），如果没有前置消息ID，则表示该消息可以消费；如果接收到含有前置消息ID的消息，我们的业务需要判断前置消息是否已经被消费，如果没有被消费那就必须等到前置消息到达后并处理完成后再消费该消息。在这个过程中业务可以临时保存该消息，也可以通知消息中间件消费失败让消息中间件重发该消息，目的就是等待前置消息的到达。

#### 顺序发送-顺序传输-顺序接收-顺序处理

这其中传输可以由`queue`来保证，但是发送接收和处理则需要应用程序来控制。简单的说，直到前一个消息发送成功，才能发送后一个消息，同样的，直到前一个消息被接受和处理结束，才能接收和处理后一个消息。这样的做法无疑会降低并发带来的好处。

以上所说的方案，是用来严格控制消息的顺序性的，然而，如果消息的发送的间隔时间足够长，不需要做过多控制，也可以控制消息的顺序性。假设，一个消息正常情况下，由发送，传输，接收到处理成功，最大时间耗费是2秒，那么只要我们保证消息发送的时间隔达不低于2秒，那么这两个消息就可以被正确的处理。这一条件咋听起来很苛刻，但事实上大部分的消息都满足了。比如，同一个订单的创建，和请求支付，最快也要数秒的时间，而退款的开始和结束，可能要数天。在典型的web应用中，操作人员不可能那么快的点击系统，而系统也不可能那么快的响应。进一步的，如果两个需要顺序处理的业务事件可能在极端的时间里连续发生，我们只要在程序上控制，人为将间隔拉开，就可以保证顺序性。

在系统的各各环节控制消息的顺序，代价高昂，而带来的好处却只是针对那极少数极端情况；通过业务或程序的方式，保证消息的有效时间间隔，代价较小，也能有效保证顺序。

如果系统中有很多的间隔极端，又需要保证顺序的消息，那你就要考虑是否将这两个消息合并成一个，或者不该采用jms了。


### activemq 消息顺序

`ActiveMQ`针对一个`topic`可以保证在只有一个发送者多个消费者的情况下消息的顺序性。 针对`queue`如果只有单个消费者和单个生产者的情况下也能保证消息的顺序性。

如果单个队列上有多个消费者，消费者会共同消费这些消息，ActiveMQ将在多个消费者之间进行负载均衡处理，因此顺序将不能得到保证。 关于该问题的背景和如何解决可以参考下面的内容：

- [独家消费者](http://activemq.apache.org/exclusive-consumer.html) 一次只有一个消费者来消费一个`queue`的消息，以此来保证消息被顺序消费（不用担心消费节点的单点问题， 你可以部署多个消费节点，activemq会自动进行容灾的。具体参考链接的内容）。
- [消息组](http://activemq.apache.org/message-groups.html) 将一个队列上的消息拆分为多个并行的虚拟排它队列，以确保到单个消息组（由JMSXGroupID头定义）的消息将保留其顺序，但不同的组将被负载均衡到不同的消费者。



### 参考

[http://sulong.me/2009/06/03/how_to_keep_jms_message_sequence](http://sulong.me/2009/06/03/how_to_keep_jms_message_sequence)
[http://activemq.apache.org/how-do-i-preserve-order-of-messages.html](http://activemq.apache.org/how-do-i-preserve-order-of-messages.html)


