# Simple Network Chat Walkthrough

### Getting a reference to ProtonManager

Before you do anything with Proton, you need a reference to ProtonManager. In your `onEnable()` method, store a reference to it.

```java
public void onEnable(){
    manager = Proton.getProtonManager();
}
```

{% hint style="info" %}
 A reference to ProtonManager will allow you to use all of Proton's API.
{% endhint %}

### Forward chat messages network-wide

Create an event listener for the async chat event and broadcast your custom data there. In this case, we opted for a container class which holds the player name and message.

```java
@EventHandler
public void onPlayerChat(AsyncPlayerChatEvent event){
    PlayerMessage message = new PlayerMessage(event.getPlayer().getName(), event.getMessage());
    manager.broadcast("networkchat", "chatMessage", message);
}
```

{% hint style="info" %}
Here we're broadcasting to the namespace **`networkchat`** with the subject **`chatMessage`**
{% endhint %}

### Listen for the broadcasted message

Here we will listen for broadcasted messages using a **`MessageHandler`**

```java
@MessageHandler(namespace = "networkchat", subject="chatMessage")
public void onChatReceived(PlayerMessage message){
    getServer().broadcastMessage(String.format("<%s> %s", message.getPlayer(), message.getMessage()));
}
```

{% hint style="info" %}
Notice: We are listening for the same namespace and subject as we sent. We are also expecting the same datatype we sent.
{% endhint %}

We must also register our **MessageHandler**s with **Proton.** This is done through the `registerMessageHandlers` method. 

```java
public void onEnable(){
    manager = Proton.getProtonManager();
    if(manager != null){
        manager.registerMessageHandlers(this, this);
    }
}
```

We're done! You will now be able to chat network-wide.

### Full Code \(with Bukkit API calls\)

{% code title="NetworkChat.java" %}
```java
public class NetworkChat extends JavaPlugin implements Listener {

    ProtonManager manager;
 
    public void onEnable(){
        manager = Proton.getProtonManager();
        if(manager != null){
            manager.registerMessageHandlers(this, this);
        }
        getServer().getPluginManager().registerEvents(this, this);
    }

    @EventHandler
    public void onPlayerChat(AsyncPlayerChatEvent event){
        PlayerMessage message = new PlayerMessage(event.getPlayer().getName(), event.getMessage());
        manager.broadcast("networkchat", "chatMessage", message);
    }

    @MessageHandler(namespace = "networkchat", subject="chatMessage")
    public void onChatReceived(PlayerMessage message){
        getServer().broadcastMessage(String.format("<%s> %s", message.getPlayer(), message.getMessage()));
    }

    public class PlayerMessage {
        String player;
        String message;

        public PlayerMessage(String player, String message) {
            this.player = player;
            this.message = message;
        }

        public String getPlayer() {
            return player;
        }
        
        public String getMessage() {
            return message;
        }
    }
}
```
{% endcode %}



