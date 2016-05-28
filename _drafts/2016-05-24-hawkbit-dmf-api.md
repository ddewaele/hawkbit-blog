---
layout: post
title: Hawkbit DMF API
date:   2016-05-26T00:09:57+02:00
categories: jekyll update
---
In this post we'll go over the Hawkbit DMF API

# Introduction

The hawkbit device simulator is a good way to see how the DMF API can be used.
Upon startup, the simular creates a number of targets.

It does so by sending an empty message with the following header

```
{sender=simulator, type=THING_CREATED, tenant=DEFAULT, thingId=simulated0}
```

And a `replyTo = simulator.replyTo` to the `dmf.exchange`.


When a distribution is launched from Hawkbit, the following messages arrives

MessageProperties [headers={topic=DOWNLOAD_AND_INSTALL, content-type=application/json, type=EVENT, tenant=DEFAULT, __TypeId__=org.eclipse.hawkbit.dmf.json.model.DownloadAndUpdateRequest, thingId=simulated0}, timestamp=null, messageId=null, userId=null, appId=null, clusterId=null, type=null, correlationId=null, replyTo=null, contentType=application/json, contentEncoding=UTF-8, contentLength=0, deliveryMode=PERSISTENT, expiration=null, priority=0, redelivered=false, receivedExchange=simulator.replyTo, receivedRoutingKey=, deliveryTag=1, messageCount=0]


{
   "actionId":1,
   "targetSecurityToken":"DZoRR5L8AASnTH1JvYcT1xaMOchxrYCH",
   "softwareModules":[
      {
         "moduleId":3,
         "moduleType":"Configuration Module",
         "moduleVersion":"1.2",
         "artifacts":[

         ]
      }
   ]
}


When the device is succesfull, 


MessageProperties [headers={topic=UPDATE_ACTION_STATUS, content-type=application/json, type=EVENT, tenant=DEFAULT}, timestamp=null, messageId=null, userId=null, appId=null, clusterId=null, type=null, correlationId=null, replyTo=null, contentType=application/octet-stream, contentEncoding=null, contentLength=0, deliveryMode=PERSISTENT, expiration=null, priority=0, redelivered=null, receivedExchange=null, receivedRoutingKey=null, deliveryTag=0, messageCount=null]

(Body:'{"actionId":1,"actionStatus":"FINISHED","message":["Simulation complete!"]}'


MessageProperties [headers={topic=UPDATE_ACTION_STATUS, content-type=application/json, type=EVENT, tenant=DEFAULT, __TypeId__=org.eclipse.hawkbit.dmf.json.model.ActionUpdateStatus}, timestamp=null, messageId=null, userId=null, appId=null, clusterId=null, type=null, correlationId=null, replyTo=null, contentType=application/json, contentEncoding=UTF-8, contentLength=75, deliveryMode=PERSISTENT, expiration=null, priority=0, redelivered=null, receivedExchange=null, receivedRoutingKey=null, deliveryTag=0, messageCount=null])


Sends it to dmf.exchange

