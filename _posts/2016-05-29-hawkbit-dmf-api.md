---
layout: post
title: Hawkbit DMF API
date:   2016-05-26T00:09:57+02:00
categories: hawkbit iot
excerpt_separator: <!--more-->
---

# Introduction 

In this post we'll go over the Hawkbit DMF API. The DMF is based on a publish-subscribe mechanism, using [RabbitMQ](https://www.rabbitmq.com) as the message broker. An in-depth look at RabbitMQ is beyond the scope of this post, but I highly recommend you checkout [the core AMPQ concepts](https://www.rabbitmq.com/tutorials/amqp-concepts.html) to get a proper understanding of exchanges, bindings and queues.

We'll focus on the interactions between the devices and the hawkbit server, and how they are implemented through the DMF API.

<!--more-->

Device will send messages to the exchange where hawkbit is listening. Hawkbit will in turn respond by sending messages to the device

# Scenarios

When performing software updates, one ore more things can happen. A software rollout can happen without any hickups, but sometimes the client will decide to abort the rollout. Alternatively, the server can also at one point decide to cancel a rollout.

Depending on the number of failures you might decide to cancel the rollout of your entire fleet.

Hawkbit provides support for all of these scenarios. Lets take a look at a couple of them.

## A typical (happy) interaction

In this scenario, everything is going fine. The device gets notified that a new software distribution is ready, it goes on to retrieve it, does the download and runs the upgrade. Everything is running fine.

- Device -> Hawkbit : device registration (hello, I'm here)
- Hawkbit -> Device : a new distribution is ready for you
- Device -> Hawkbit  : Thanks, going to retrieve the software updates
- Device -> Hawkbit  : Download is finished and everything is ok.
- Hawkbit  : Marks the distribution as ok (in-active)

![]({{ site.url }}/assets/images/dmf/hawkbit-happy-flow.png)

There's not much to say about this scenario except that this is what we would always like to see. Unfortunately this is not always the case.

## A (non-happy) scenario 1

In this scenario the device flags an error during the software update process. It needs to inform the hawkbit server about this error so that hawkbit can tak appropriate action. Hawkbit will cancel the rollout for this device and mark the distribution as in-active, but it might also decide to cancel the entire rollout of all devices, based on the number of other devices that experienced the same error.

- Device -> Hawkbit : device registration (hello, I'm here)
- Hawkbit -> Device : a new distribution is ready for you
- Device -> Hawkbit  : Thanks, going to retrieve the software updates
- Device -> Hawkbit  : Thanks, but I'm having some trouble right now, this download might now work.
- Device -> Hawkbit  : Nope, I was unable to download the firmware sorry. 
- Hawkbit  : Marks the distribution as error (in-active)

![]({{ site.url }}/assets/images/dmf/hawkbit-unhappy-flow1.png)

## A (non-happy) scenario 2

It's also possible that hawkbit itself will decide to abort a software rollout. When it does that, certain devices might alreay be busy doing the upgrade. So hawkbit will send a message to them informing them that they should cancel the software rollout.
The device will need to confirm to hawkbit that they indeed received the cancelation request and that they either confirmed that request, or rejected it.

- Device -> Hawkbit : device registration (hello, I'm here)
- Hawkbit -> Device : a new distribution is ready for you
- Device -> Hawkbit  : Thanks, going to retrieve the software updates
- Hawkbit -> Device  : Wait ... sorry, need to cancel the rollout.
- Device -> Hawkbit  : OK, confirming the cancelation ... better luck next time.

![]({{ site.url }}/assets/images/dmf/hawkbit-unhappy-flow2.png)


# Message interactions

In this section, we'll go over the different message interactions in more detial, showing you 

- who sends the messages
- to what exchange messages get sent
- what the headers / properties / payload of each message is


## Device registration

When a new device is bootstrapped in the field, it can register itself with the hawkbit server.
This is an optional step, as another scenario might involve the device registration already having been performed by another component within your architecture.

Anyways, Hawkbit does provide an API to register devices through DMF.

In order to register a device, Hawkbit expects an empty message to be sent to the `dmf.exchange` with a header containing the following key/value pairs:

{:.foo}
|Key | Value |
| :---: | :---: |
| sender | simulator |
| type | THING_CREATED |
| tenant | DEFAULT |
| thingId | new_device_1 |

The device can also set a `replyTo` header to ensure that it receives feedback from Hawkbit on that particular address.

You'll see that the device will be created:

![]({{ site.url }}/assets/images/dmf/1-no-distribution.png)


If you have the `rabbitmqadmin` script installed, you can execute the following command from the command line also:

```shell
./rabbitmqadmin publish routing_key='' exchange='dmf.exchange' payload='' properties='{"content_type": "application/json","reply_to":"cli","headers":{"type":"THING_CREATED", "tenant":"DEFAULT","thingId":"new_device_1"}}' payload_encoding="string"
```

Or you can use one of the many [RabiitMQ client libraries](https://www.rabbitmq.com/devtools.html) available.

```java
package com.ecs.rabbitmq.java.client;

import java.util.HashMap;
import java.util.Map;

import com.rabbitmq.client.AMQP.BasicProperties;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class RegisterDevice {

   public static void main(String[] argv) throws Exception {

      ConnectionFactory factory = new ConnectionFactory();
      factory.setHost("localhost");
      
      Connection connection = factory.newConnection();
      Channel channel = connection.createChannel();
      
      Map<String,Object> headers = new HashMap<>();
      
      headers.put("sender","simulator");
      headers.put("type","THING_CREATED");
      headers.put("tenant","DEFAULT");
      headers.put("thingId","new_device");
      
      BasicProperties props = new BasicProperties
                .Builder()
                .replyTo(Constants.JAVA_CLIENT_REPLYTO)
                .headers(headers)
                .contentType("application/json")
                .build();
      
      channel.basicPublish(Constants.DMF_EXCHANGE, "",props,null);
      
      channel.close();
      connection.close();
   }
}
```

## Setting up a distribution

The main goal of Hawkbit is to facilitate software distribution. So the next step is to assign a software distribution to a device (by dragging the software distribution onto our newly created device).

![]({{ site.url }}/assets/images/dmf/2-distribution-assigned.png)

The device will notified of this as Hawkbit will send a `DOWNLOAD_AND_INSTALL` event message to the address specified in the `replyTo` header.

The message will look like this:

```json
{
   "actionId":5,
   "targetSecurityToken":"i1irHaaUJNHZDXTIQ4o2KUsG3J2TN9Qf",
   "softwareModules":[
      {
         "moduleId":3,
         "moduleType":"Configuration Module",
         "moduleVersion":"1.2",
         "artifacts":[
            {
               "filename":"emptyconfig.xml",
               "hashes":{
                  "sha1":"680f4b06424f4c4e12b52e5f85c5903c13082d3c",
                  "md5":"1896f4052d3089ae5524dc2de823bb78"
               },
               "size":10,
               "urls":{
                  "HTTP":"http://localhost:8080/DEFAULT/controller/v1/new_device/softwaremodules/3/artifacts/emptyconfig.xml"
               }
            }
         ]
      }
   ]
}
```

The following headers will also be sent to the device.


|Key | Value |
| :---: | :---: |
| topic | DOWNLOAD_AND_INSTALL |
| content-type | application/json |
| type | EVENT |
| tenant | DEFAULT |
| __TypeId__ | org.eclipse.hawkbit.dmf.json.model.DownloadAndUpdateRequest |
| thingId | new_device |


## Acknowledging the distribution

At this point the device knows that a software distribution has been setup for it. The device can respond to it by sending one or more `org.eclipse.hawkbit.dmf.json.model.ActionUpdateStatus` messages for this particular distribution action.

It needs to send a message to Hawkbit with one of the following statuses :

- DOWNLOAD
- RETRIEVED
- RUNNING
- FINISHED
- ERROR
- WARNING
- CANCELED
- CANCEL_REJECTED

Lets take a look at some of them

## ActionUpdateStatus

`ActionUpdateStatus` message are really simple status messages that the device will send to hawkbit to keep it up to date on the software distribution set rollout.

The message adds a `topic=UPDATE_ACTION_STATUS` property to it to identify it as a status message.

|Key | Value |
| :---: | :---: |
| topic | UPDATE_ACTION_STATUS |
| content-type | application/json |
| type | EVENT |
| tenant | DEFAULT |


The payload itself is really simple, and uses the `actionId` to identify the software distribution for that device

```json
{
   "actionId":1,
   "actionStatus":"DOWNLOAD",
   "message":[
      "Downloading the distribution"
   ]
}
```

All of these status messages can be tracked in the UI, in the Action History view of the software distribution set.

![]({{ site.url }}/assets/images/dmf/6-finished-distribution.png)

Once the software distribution reaches the FINISHED state, it will be marked "done" and set inactive.


The device itsel may also decide to finished the software distribution with an `ERROR`. In that case hawkbit will also mark the software distribution inactive.

![]({{ site.url }}/assets/images/dmf/8-finished-distribution-with-error.png)


## Hawkbit cancelling the distribution

It's possible for one reason or another that Hawkbit itself will attempt to cancel the distribution. This can be done in the UI by right-clicking on the distribution history and clicking Cancel.

The device will be notified of that with the following event

|Key | Value |
| :---: | :---: |
| topic | CANCEL_DOWNLOAD |
| content-type | application/json |
| type | EVENT |
| tenant | DEFAULT |
| __TypeId__ | Long |
| thingId | new_device_1 |


It is now given the option to either

- confirm the cancelation
- reject the cancel.

A force quit does not generate an action to the client

When the device is succesfull, 

MessageProperties [headers={topic=UPDATE_ACTION_STATUS, content-type=application/json, type=EVENT, tenant=DEFAULT}, timestamp=null, messageId=null, userId=null, appId=null, clusterId=null, type=null, correlationId=null, replyTo=null, contentType=application/octet-stream, contentEncoding=null, contentLength=0, deliveryMode=PERSISTENT, expiration=null, priority=0, redelivered=null, receivedExchange=null, receivedRoutingKey=null, deliveryTag=0, messageCount=null]

{"actionId":1,"actionStatus":"FINISHED","message":["Simulation complete!"]}


MessageProperties [headers={topic=UPDATE_ACTION_STATUS, content-type=application/json, type=EVENT, tenant=DEFAULT, __TypeId__=org.eclipse.hawkbit.dmf.json.model.ActionUpdateStatus}, timestamp=null, messageId=null, userId=null, appId=null, clusterId=null, type=null, correlationId=null, replyTo=null, contentType=application/json, contentEncoding=UTF-8, contentLength=75, deliveryMode=PERSISTENT, expiration=null, priority=0, redelivered=null, receivedExchange=null, receivedRoutingKey=null, deliveryTag=0, messageCount=null])


Sends it to dmf.exchange

