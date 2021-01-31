---
description: >-
  A guide that should kickstart your usage of Proton. Prefer to go through an
  example? Check out the global chat walkthrough.
---

# Getting Started

### Proton as a Dependency

#### Using Maven

```markup
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>
```

```markup
<dependency>
    <groupId>com.github.mcgrizzz</groupId>
    <artifactId>Proton</artifactId>
    <version>v1.2.1</version>
    <scope>provided</scope>
</dependency>
```

#### Using Gradle

```groovy
repositories {
    ...
    maven { url 'https://jitpack.io' }
}
```

```groovy
dependencies {
        implementation 'com.github.mcgrizzz:Proton:v1.2.1'
}
```

## Setting-up

### Setting-up RabbitMQ

If you want to use Proton with RabbitMQ, you will first need to set up an instance of RabbitMQ.

You can do this in one of two ways:

**1. Host it yourself**

Here is a great guide to setting it up yourself: [https://www.rabbitmq.com/download.html](https://www.rabbitmq.com/download.html)

You will most likely need to setup a new username and password for your rabbitMQ instance: [https://www.rabbitmq.com/access-control.html\#user-management](https://www.rabbitmq.com/access-control.html#user-management)

This is for two reasons. One, RabbitMQ restricts the access of the default guest account only to localhost connections. Two, leaving the default username/password leaves you open to attacks.

**2. Find an online host**

Here are some hosts for RabbitMQ that you may find helpful. One of them actually has a free tier that may fit your needs.

1. [Stackhero](https://www.stackhero.io/en/services/RabbitMQ#pricing) - No free tiers but the lowest tier can probably support up to 200 servers
2. [CloudAMQP](https://www.cloudamqp.com/plans.html) - Expensive dedicated servers but there are cheaper and free shared servers.

\*When choosing a host and plan consider your network's size and needs.

### Setting-up Redis

{% hint style="danger" %}
This section is under-construction. Please take a look at [Redis Quickstart](https://redis.io/topics/quickstart)
{% endhint %}

### Proton's `config.yml` 

Before integrating with Proton, you should configure your servers' Proton configs.

Let's take a look at the config.

```yaml
rabbitMQ:
  useRabbitMQ: true
  host: "localhost"
  virtualHost: '/'
  port: 5672
  authorization:
    useAuthorization: true
    username: guest
    password: guest
redis:
  useRedis: false
  host: "localhost"
  port: 6379
  usePassword: true
  password: "password"
identification:
  clientName: "client1"
  groups: []
bStatsEnabled: true
checkForUpdates: true
```

There are a lot of options here, but let's look at them in smaller pieces.

The first section is for the configuration of RabbitMQ.

`useRabbitMQ` sets whether Proton will try to use RabbitMQ `host` is the ip of your RabbitMQ instance.  
`port` is its port which you will probably not need to change.  
`virtualHost` is the virtual host which you can think of as an extension of the host. If you want to learn more about this [there's a great writeup here](https://www.rabbitmq.com/vhosts.html).  
`useAuthorization` sets whether Proton should try to authorize the connection  
`username` is the username for the connection  
`password` is the password for the connection

```yaml
rabbitMQ:
  useRabbitMQ: true
  host: "localhost"
  virtualHost: '/'
  port: 5672
  authorization:
    useAuthorization: true
    username: guest
    password: guest
```

Next we have the `Redis` section of the config.

`useRedis` sets whether Proton will try to use Redis. \(NOTE: Proton only uses one service at a time, if both are selected, Proton will use RabbitMQ\)  
`host` is the ip of your Redis instance.  
`port` is its port  
`usePassword` sets whether Proton will try to use a password for the connection  
`useAuthorization` is the password Proton will try to use.  
`username` is the username for the connection  
`password` is the password for the connection

```yaml
redis:
  useRedis: false
  host: "localhost"
  port: 6379
  usePassword: true
  password: "password"
```

Lastly, we have the `identification` section of the config. This is what Proton uses to know which servers are which.

{% hint style="info" %}
`clientName` should be a unique name to your server. No two servers should have the same name. This is important if you want to know later on which server a message came from.
{% endhint %}

`groups` is a list of groups that the current server belongs to. For example, you can have a server that belongs to the group 'hub'. Therefore, when you send a message to the hub group, only servers in that group will receive that message. You can leave groups empty if you don't need it.

```yaml
identification:
  clientName: "client1"
  groups: []
```

`bStatsEnabled` is a boolean value that enables basic metric collection.

`checkForUpdates` is a boolean value that enables update checks which will run only once, on server startup.

### Your plugin

Just make sure that in your `plugin.yml`, you include the dependency for Proton.

```yaml
main: org.test.Test
name: TestPlugin
version: 1.0
depend:
  - Proton
```

## Proton Usage

### Getting an instance to `ProtonManager`

To start using Proton, you should first get an instance of `ProtonManager`.

```java
private ProtonManager protonManager;

@Override
public void onEnable() {
    this.protonManager = Proton.getProtonManager();
}
```

`ProtonManager` should not be `null` at this point. If it is, check your console for connection and configuration errors.

### Sending your first message

Now that you have a reference to `ProtonManager`, you can send your first message.

```java
String namespace = "namespace";
String subject = "subject";
String recipient = "recipient";
Object data = new Object();
protonManager.send(namespace, subject, data, recipient);
```

Let's break down these arguments.

* `namespace` identifies your organization or plugin. This is what keeps your messages within the scope of your plugin or organization
* `subject` is used to identify the type of message you're sending, you can put any value, but we recommend something relevant and descriptive.
* `recipient` is used to define the client or group you wish to send to.
* `data` is the object that you wish to send.

{% hint style="info" %}
The **`data`** you send can be any object or primitive. The only caveat is that is that is must be Json serializable. Otherwise, you will receive exceptions.
{% endhint %}

If you want to send a message to all clients that may be listening to a specific `namespace` and `subject` you can use the broadcast method instead:

```java
String namespace = "myPluginOrOrganization";
String subject = "subjectOfMyMessage";
Object data = new Object();
protonManager.broadcast(namespace, subject, data);
```

{% hint style="info" %}
**`namespace`** and **`subject`**form what is called a **`MessageContext`**. Each `MessageContext` can only have one defined datatype. So if you define a namespace and subject, make sure you always send the same type of data through that context.
{% endhint %}

{% hint style="danger" %}
The use of **`.`** \(period\) is not allowed when defining a namespace, subject, recipient, or group. It is a reserved character used for internal processing.
{% endhint %}

### Receiving a message

We tried to model the message receive system similarly to the Event system you probably use regularly.

In any class or object, you can define a `MessageHandler`. A `MessageHandler` is an annotated method which receives data for a specific `MessageContext`.

Let's take a look at the receiving end of the message sent above.

```java
class MyClass {
    ...
    @MessageHandler(namespace="namespace", subject="subject")
    public void anyMethodName(Object data){
        //do something with the data received
    }
    ...
}
```

If you want to know the sender of the message, you can attach a second parameter to your `MessageHandler` method.

```java
class MyClass {
    ...
    @MessageHandler(namespace="namespace", subject="subject")
    public void anyMethodName(Object data, MessageAttributes attr){
        String senderName = attr.getSenderName();
        UUID senderID = attr.getSenderID();
    }
    ...
}
```

The code within a `MessageHandler` is synchronous with Bukkit by default. This was a design decision to match the fact that most API calls must be synchronous. However, you can receive messages asynchronously if you wish by adding an optional attribute.

```java
@MessageHandler(namespace="namespace", subject="subject", async=true)
```

The final step to actual receive any messages, is to register your `MessageHandler(s)`. Similarly to the Event API, you just register your class instance with the `ProtonManager`.

```java
@Override
public void onEnable() {
    this.protonManager = Proton.getProtonManager();
    if(this.protonManager != null){
        this.protonManager.registerMessageHandlers(this, new MyClass());   
    }
}
```

If you want, you can register all of your handlers in one call.

```java
this.protonManager.registerMessageHandlers(this, handler1, handler2, handler3...);
```

If you have any lingering questions, feel free to consult [the examples repo](https://github.com/mcgrizzz/ProtonExamples). You can also submit `question` issue [here](https://github.com/mcgrizzz/Proton/issues).

