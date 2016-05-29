---
layout: post
title: RabbitMQ tutorial
date:   2016-05-26T00:09:57+02:00
---


## CLI

https://www.rabbitmq.com/management-cli.html


rabbitmqadmin publish routing_key='' exchange='dmf.exchange' payload='{"actionId":8,"softwareModuleId":1,"actionStatus":"RETRIEVED","message":["Retrieved the distribution config"]}' properties='{"content-type":"application/json", "headers":{"topic":"UPDATE_ACTION_STATUS", "type":"EVENT", "tenant":"DEFAULT"}}'


 [x] Sent '{"actionId":8,"softwareModuleId":1,"actionStatus":"RETRIEVED","message":["Retrieved the distribution config"]}' with headers {topic=UPDATE_ACTION_STATUS, type=EVENT, tenant=DEFAULT}




./rabbitmqadmin publish routing_key='' exchange='dmf.exchange' payload='{"actionId":8,"softwareModuleId":1,"actionStatus":"RETRIEVED","message":["Retrieved the distribution config"]}' properties='{"content_type": "application/json","headers":{"topic":"UPDATE_ACTION_STATUS", "type":"EVENT", "tenant":"DEFAULT"}}' payload_encoding="string"




## Exchanges
```
./rabbitmqctl list_exchanges



curl -i -u guest:guest -H "content-type:application/json" -X POST http://localhost:15672/api/exchanges/%2f/dmf.exchange/publish -d'{"properties":{},"routing_key":"","payload":"Your payload","payload_encoding":"string"}'

amq.direct					direct
dmf.exchange				fanout
amq.fanout					fanout
amq.match					headers
amq.headers					headers
							direct
amq.rabbitmq.trace			topic
dmf.connector.deadletter	fanout
amq.topic					topic
amq.rabbitmq.log			topic
```

## Bindings

./rabbitmqctl list_bindings

								exchange	dmf_connector_deadletter_ttl	queue	dmf_connector_deadletter_ttl	[]
								exchange	dmf_receiver					queue	dmf_receiver					[]
dmf.connector.deadletter		exchange	dmf_connector_deadletter_ttl	queue									[]
dmf.exchange					exchange	dmf_receiver					queue									[]


## Queues
```
./rabbitmqctl list_queues
Listing queues ...

dmf_receiver					0
dmf_connector_deadletter_ttl	0
```

As soon as you subscribe to a topic

```java
public class ReceiveHawkbitMsgs {
  private static final String EXCHANGE_NAME = "dmf.exchange";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "fanout",true);
    String queueName = channel.queueDeclare().getQueue();
    channel.queueBind(queueName, EXCHANGE_NAME, "");

    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    Consumer consumer = new DefaultConsumer(channel) {
      @Override
      public void handleDelivery(String consumerTag, Envelope envelope,
                                 AMQP.BasicProperties properties, byte[] body) throws IOException {
        String message = new String(body, "UTF-8");
        System.out.println(" [x] Received '" + message + "' with properties " +properties);
      }
    };
    channel.basicConsume(queueName, true, consumer);
  }
}
````

You'll notice that a queue will be created 
```
./rabbitmqctl list_queues

amq.gen-GgMzd8av7QUuJdxAhOtcMA	0
dmf_receiver					0
dmf_connector_deadletter_ttl	0
```

through a binding on the exchange

````
./rabbitmqctl list_bindings

							exchange	amq.gen-GgMzd8av7QUuJdxAhOtcMA	queue	amq.gen-GgMzd8av7QUuJdxAhOtcMA	[]
							exchange	dmf_connector_deadletter_ttl	queue	dmf_connector_deadletter_ttl	[]
							exchange	dmf_receiver					queue	dmf_receiver					[]
dmf.connector.deadletter	exchange	dmf_connector_deadletter_ttl	queue									[]
dmf.exchange				exchange	amq.gen-GgMzd8av7QUuJdxAhOtcMA	queue									[]
dmf.exchange				exchange	dmf_receiver					queue									[]
```