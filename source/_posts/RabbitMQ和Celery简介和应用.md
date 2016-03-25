title: RabbitMQ和Celery简介和应用
date: 2016-03-25 03:42:23
tags:
---

## 简介

在我们的OJ中，大量的异步操作都使用消息队列来（Message Queue 简称 MQ）传递信息，我们主要使用celery 执行异步任务，比如代理题目提交等功能，也有的地方直接使用 RabbitMQ 进行通信，比如我们的[分布式判题节点](https://github.com/NJUST-FishTeam/OnlineJudgeSite_M6)

由于使用了消息对了，我们可以很轻松的增加评测机的数目，只需要在新的虚拟机中启动判题节点，让判题节点连接消息队列自己接受任务即可。同时，消息队列具有持久化功能，如果OJ中的某个组件出现故障，其他部件发出的消息并不会丢失，而是等待故障部件恢复之后再消费信息。

RabbitMQ 是实现了 **AMQP** ([Advanced_Message_Queuing_Protocol](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol))的消息队列，使用Erlang编写。

Celery 是一个 Python 分布式的任务队列，主要用于完成异步任务，可以选择多种后端(Broker),例如RabbitMQ， Redis， 甚至数据库都可以。虽然主要用于完成异步任务，但是也可以用于完成定时任务和直接从RabbitMQ中接收消息。

## RabbitMQ 可视化管理
RabbitMQ 提供一个HTTP的可视化管理插件，可以通过浏览器进行方便的监视和管理操作，使用方法

> $ rabbitmq-plugins enable rabbitmq_management

重启RabbitMQ， 在15672端口即可进行管理，默认用户名密码均为guest

参见 [RabbitMQ Management Plugin](https://www.rabbitmq.com/management.html)

## Queue， Exchange，Routing_key

Queue（队列），Exchang（交换器），Routing_key（路由关键字），都是AMQP协议的重要组重部分，要谈RabbitMQ的使用，不得不说这三个概念。

首先让我们来看看一条消息是如何由生产者（Producer）通过 MQ 传递给消费者（Consumer）:

1. 生产者的消息上必须有一个 routing_key
2. 生产者决定将消息发送到哪个 Exchange
3. Queue 绑定（Bind） 到若干个Exchange上，Queue 绑定的对象是一个二元组（Exchange，binding_key）
4. Exchange 接到 Message ，用routing_key 和 binding_key 决定将消息发往哪个队列（Queue）
5. 消费者绑定消费消息，如果有必要回复ack


Queue，是AMQP中消费者最直接的消费对象，一个队列可以有多个消费者消费。

Exchange， 消息生产者将消息交给Exchange ， Exchange 根据 Bind 关系将消息分发给Queue。

Exchange 有多种类型，最常见的是direct， 就是当routing_key == binding_key 的时候，转发给Queue。

## RabbitMQ 直接使用方法

**发送消息**

```python
connection = pika.BlockingConnection(pika.ConnectionParameters(host="localhost"))
channel = connection.channel()
channel.queue_declare(queue="judge_task", durable=True) # 声明名为"judge_task" 的持久化队列， 如果RabbitMQ中本来就存在队列，则不用建立，非必须
channel.basic_publish(exchange=settings.JUDGESITE_SETTINGS['judge_exchange'],
                         routing_key="judge_task",
                         body=json.dumps(body, ensure_ascii=False),
                         properties=pika.BasicProperties(
                              delivery_mode=2,  # make message persistent
    ))
```

**发送消息**

```python
self.connection = pika.BlockingConnection(pika.ConnectionParameters(host="localhost"))
self.channel = connection.channel()
self.channel.queue_declare(queue="judge_task", durable=True)
self.channel.queue_bind(queue="judge_task",
                   exchange="judge_exchange",
                   routing_key="judge_task") # 将队列和Exchange 绑定，使用 routingkey
self.channel.basic_qos(prefetch_count=1) # 每次接受一个消息
self.channel.basic_consume(self._consume, queue="judge_task") # _consume 是消费函数
```
_consume 函数样例
```python
def _consume(self, ch, method, properties, body):
	logging.info("GOT A TASK!")
    print body
    self.channel.basic_ack(delivery_tag=method.delivery_tag)
    logging.info("TASK IS DONE!")
```

## 使用 celery 直接接收 RabbitMQ 消息

celery 可以直接作为 RabbitMQ 消息的消费者，参见[Can a celery worker/server accept tasks from a non celery producer?](http://stackoverflow.com/questions/11964742/can-a-celery-worker-server-accept-tasks-from-a-non-celery-producer/34963811#34963811)

```python
from celery import Celery
from celery import bootsteps
from kombu import Consumer, Exchange, Queue

my_queue = Queue('custom', Exchange('custom'), 'routing_key') # 三个参数分别为队列名，Exchange， routing_key

app = Celery(broker='amqp://')


class MyConsumerStep(bootsteps.ConsumerStep):

    def get_consumers(self, channel):
        return [Consumer(channel,
                         queues=[my_queue],
                         callbacks=[self.handle_message],
                         accept=['json'])]

    def handle_message(self, body, message):
        print('Received message: {0!r}'.format(body))
        message.ack()
app.steps['consumer'].add(MyConsumerStep)
```