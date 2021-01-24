# Introduction

{% page-ref page="./" %}

## Proton

Proton is a library which aims to give you a reliable and flexible solution to cross-server messaging. It uses your choice of Redis or RabbitMQ. Other methods, such as Plugin Messaging, are either difficult and messy to implement or have restrictions such as the inability to send messages to a server with no active players. Proton is different in that it

1. creates a simple system for messaging between servers and
2. is robust and versatile enough to where you can implement any messaging need you require.

 Proton is still being actively developed and tested. Your feedback is welcome.

#### What is RabbitMQ?

 RabbitMQ is a queue based messaging broker. In its simplest form, a producer sends a message to a queue, then a consumer consumes that message from the queue. However, RabbitMQ can and usually does support more complex networks than that. Proton acts as an interface between your plugin and the client API for RabbitMQ. RabbitMQ can be hosted easily on your own servers or by a cloud provider. [You can read more here.](https://www.rabbitmq.com/#getstarted)

#### What is Redis?

Redis a in-memory data structure store which is often used as a database or message broker. While it is not solely used for brokering messages, it is very fast and is a certainly a good choice for many situations. [You can read more here.](https://redis.io/)

