---
layout: post
title:  "First look at Eclipse IOT Hawkbit"
date:   2016-05-21T00:09:57+02:00
categories: jekyll update
---

# Introduction

In this post I'm going to take a closer look at Hawkbit.

# Getting started

To get started we need to get a hold of the code, build it and then run.

## Clone the repo and build

The code is in the following github repository and is built using maven.

{% highlight shell %}
git clone https://github.com/eclipse/hawkbit.git
cd hawkbit
mvn clean install
{% endhighlight %}

This will build the entire codebase.

## Start the dependencies

### MongoDB

MongoDB is used to store all software artifacts.

- Locate your `mongodb-osx-x86_64` distribution
- Create the data-files if the don't exist yet.
- Start mongo db

{% highlight shell %}
cd /Users/ddewaele/Projects/iot/mongodb-osx-x86_64-3.0.12/bin
mkdir ../data-files
./mongod --dbpath ../data-files

2016-05-24T00:00:28.181+0200 I CONTROL  [initandlisten] MongoDB starting : pid=6952 port=27017 dbpath=../data-files 64-bit host=MacBook-Pro-3.local
2016-05-24T00:00:28.181+0200 I CONTROL  [initandlisten] db version v3.0.12
2016-05-24T00:00:28.181+0200 I CONTROL  [initandlisten] git version: 33934938e0e95d534cebbaff656cde916b9c3573
2016-05-24T00:00:28.181+0200 I CONTROL  [initandlisten] build info: Darwin mci-osx1010-20.build.10gen.cc 14.0.0 Darwin Kernel Version 14.0.0: Fri Sep 19 00:26:44 PDT 2014; root:xnu-2782.1.97~2/RELEASE_X86_64 x86_64 BOOST_LIB_VERSION=1_49
2016-05-24T00:00:28.181+0200 I CONTROL  [initandlisten] allocator: system
2016-05-24T00:00:28.181+0200 I CONTROL  [initandlisten] options: { storage: { dbPath: "../data-files" } }
2016-05-24T00:00:28.190+0200 I JOURNAL  [initandlisten] journal dir=../data-files/journal
2016-05-24T00:00:28.190+0200 I JOURNAL  [initandlisten] recover : no journal files present, no recovery needed
2016-05-24T00:00:28.204+0200 I JOURNAL  [durability] Durability thread started
2016-05-24T00:00:28.204+0200 I JOURNAL  [journal writer] Journal writer thread started
2016-05-24T00:00:28.204+0200 I CONTROL  [initandlisten] 
2016-05-24T00:00:28.205+0200 I CONTROL  [initandlisten] ** WARNING: soft rlimits too low. Number of files is 256, should be at least 1000
2016-05-24T00:00:28.208+0200 I INDEX    [initandlisten] allocating new ns file ../data-files/local.ns, filling with zeroes...
2016-05-24T00:00:28.238+0200 I STORAGE  [FileAllocator] allocating new datafile ../data-files/local.0, filling with zeroes...
2016-05-24T00:00:28.238+0200 I STORAGE  [FileAllocator] creating directory ../data-files/_tmp
2016-05-24T00:00:28.403+0200 I STORAGE  [FileAllocator] done allocating datafile ../data-files/local.0, size: 64MB,  took 0.165 secs
2016-05-24T00:00:28.427+0200 I NETWORK  [initandlisten] waiting for connections on port 27017
{% endhighlight %}

### RabbitMQ


- Locate your rabbitmq installation
- Launch rabbitmq
- Install the [RabbitMQ management plugin](https://www.rabbitmq.com/management.html)


{% highlight shell %}
cd /Users/ddewaele/Projects/iot/rabbitmq_server-3.6.1/sbin/
rabbitmq-plugins enable rabbitmq_management
./rabbitmq-server 

              RabbitMQ 3.6.1. Copyright (C) 2007-2016 Pivotal Software, Inc.
  ##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
  ##  ##
  ##########  Logs: /Users/ddewaele/Projects/iot/rabbitmq_server-3.6.1/var/log/rabbitmq/rabbit@MacBook-Pro-3.log
  ######  ##        /Users/ddewaele/Projects/iot/rabbitmq_server-3.6.1/var/log/rabbitmq/rabbit@MacBook-Pro-3-sasl.log
  ##########
              Starting broker... completed with 6 plugins.
{% endhighlight %}


## Run the applications

{% highlight shell %}
java -jar ./examples/hawkbit-example-app/target/hawkbit-example-app-0.2.0-SNAPSHOT.jar
java -jar ./examples/hawkbit-device-simulator/target/hawkbit-device-simulator-0.2.0-SNAPSHOT.jar
java -jar ./examples/hawkbit-mgmt-api-client/target/hawkbit-mgmt-api-client-0.2.0-SNAPSHOT.jar
{% endhighlight %}


# The model

The basic idea around Hawkbit is software distribution. How do we manage that process and how can we get software artifacts onto devices. Hawkbit has create a model to support this concept.

Everything starts with the concept of a software module.
A software module is an artifact of a specific software module type. 
A software module type defines if its software or firmware.

Example software module types can be 

- Base OS Image
- Software Module
- Configuration Module

![](/assets/images/ps_neutral.png)

In the screenshot below, you can see the 3 types :

![](/assets/images/upload_mgmt.png)


# API Integration

## DDI API

### Retrieving controller info


Every controller can retrieve its information from the hawkbit server like this

```
curl -v curl -v http://localhost:8080/default/controller/v1/controller1 | python -m json.tool
```

This will return a number of links that the controller can follow, as well as some configuration.

```json
{
    "_links": {
        "configData": {
            "href": "http://localhost:8080/default/controller/v1/controller1/configData"
        }
    },
    "config": {
        "polling": {
            "sleep": "00:05:00"
        }
    }
}
```


When we assign a distribution to a device, a deploymentBase link will become visibe.

```json
{
    "_links": {
        "configData": {
            "href": "http://localhost:8080/default/controller/v1/controller1/configData"
        },
        "deploymentBase": {
            "href": "http://localhost:8080/default/controller/v1/controller1/deploymentBase/4?c=160000543"
        }
    },
    "config": {
        "polling": {
            "sleep": "00:05:00"
        }
    }
}
```

This `deploymentBase` link will give access to various `chunks` in the deployment. Each `chunck` contains zero or more artifacts that can be downloaded by the device:

```
curl http://localhost:8080/default/controller/v1/controller1/deploymentBase/4?c=159999582 | python -m json.tool
```

```json
{
    "deployment": {
        "chunks": [
            {
                "artifacts": [
                    {
                        "_links": {
                            "download-http": {
                                "href": "http://localhost:8080/default/controller/v1/controller1/softwaremodules/1/artifacts/emptyfirmware.bin"
                            },
                            "md5sum-http": {
                                "href": "http://localhost:8080/default/controller/v1/controller1/softwaremodules/1/artifacts/emptyfirmware.bin.MD5SUM"
                            }
                        },
                        "filename": "emptyfirmware.bin",
                        "hashes": {
                            "md5": "b026324c6904b2a9cb4b88d6d61c81d1",
                            "sha1": "e5fa44f2b31c1fb553b6021e7360d07d5d91ff5e"
                        },
                        "size": 2
                    }
                ],
                "name": "IxorTalk Base Image",
                "part": "Base OS Image",
                "version": "1"
            },
            {
                "artifacts": [
                    {
                        "_links": {
                            "download-http": {
                                "href": "http://localhost:8080/default/controller/v1/controller1/softwaremodules/3/artifacts/emptyconfig.xml"
                            },
                            "md5sum-http": {
                                "href": "http://localhost:8080/default/controller/v1/controller1/softwaremodules/3/artifacts/emptyconfig.xml.MD5SUM"
                            }
                        },
                        "filename": "emptyconfig.xml",
                        "hashes": {
                            "md5": "1896f4052d3089ae5524dc2de823bb78",
                            "sha1": "680f4b06424f4c4e12b52e5f85c5903c13082d3c"
                        },
                        "size": 10
                    }
                ],
                "name": "IxorTalk Customer1 Config",
                "part": "Configuration Module",
                "version": "3"
            },
            {
                "artifacts": [
                    {
                        "_links": {
                            "download-http": {
                                "href": "http://localhost:8080/default/controller/v1/controller1/softwaremodules/2/artifacts/emptysw.bin"
                            },
                            "md5sum-http": {
                                "href": "http://localhost:8080/default/controller/v1/controller1/softwaremodules/2/artifacts/emptysw.bin.MD5SUM"
                            }
                        },
                        "filename": "emptysw.bin",
                        "hashes": {
                            "md5": "2a5ea88c94ba0193bd1fd95c645eb557",
                            "sha1": "d94eeb5ca871d4c56a44f9d96cc3e1b2992d06cc"
                        },
                        "size": 8
                    }
                ],
                "name": "IxorTalk Customer1 Software",
                "part": "Software Module",
                "version": "2"
            }
        ],
        "download": "forced",
        "update": "forced"
    },
    "id": "4"
}
```



curl -v curl -v http://localhost:8080/default/controller/v1/controller1 | python -m json.tool
curl -v -H "Content-Type: application/json" -d @progress1.json  -X POST http://localhost:8080/default/controller/v1/controller1/deploymentBase/3/feedback
curl -v http://localhost:8080/default/controller/v1/controller1/deploymentBase/13?c=160267701 | python -m json.tool
wget http://localhost:8080/default/controller/v1/controller1/softwaremodules/7/artifacts/emptyfirmware.bin.MD5SUM
wget http://localhost:8080/default/controller/v1/controller1/softwaremodules/8/artifacts/emptysw.bin



Here you can see th


rabbitmqadmin publish routing_key=test  exchange=dmf.exchange payload='{"type":"THING_CREATED","tenant":"tenant123","thingId":"abc","sender":"Lwm2m"}' properties='{"type":"THING_CREATED","tenant":"tenant123","thingId":"abc","sender":"Lwm2m"}'


rabbitmqadmin get queue=test requeue=false
rabbitmqadmin publish routing_key=test exchange=dmf.exchange payload="hello"


# Issues

- JSON date handling (ISO-8601)
- API versioning (allowing the APIs to evolve)
- Client APIs (Feign) - use proper Jackson/GSON decoders instead of working with strings, allow clients to specify more parameters.
- ControllerResource returns strings. Shouldn't we use the objects fro tis ?

                    final String deploymentJson = controllerResource.getDeployment(getTenant(), getId(), actionId);
                    final String swVersion = JsonPath.parse(deploymentJson).read("deployment.chunks[0].version");

# Remarks

- Cancelled icon is a green circle with white x. I would change the green color as it resembles finished icon a lot.


# Questions

- How does hawkbit offer secure artifact downloads ? Currently a client can download any software module offered by hawkbit without providing credentials. I noticed some references to hawkbit.server.security.* properties but dont know if they are actually used yet.
(I also saw some kind of GatewayToken being used in the simulator).

- Plans on integration with other repositories ? (ex: Nexus / Artifactory / npm)

- Plans to support different Hawkbit clients ? (Java / C / NodeJS / ....)


# References

[Eclipse Project Page](https://projects.eclipse.org/proposals/hawkbit)
[Hawkbit EclipseCon presentation](http://sp-collateral.apps.bosch-iot-cloud.com/slides/eclipseConNA2016.html#/)
[](https://jaxenter.com/eclipse-hawkbit-126445.html)

