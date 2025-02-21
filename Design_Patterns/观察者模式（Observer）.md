**定义：**

​		观察者模式（又被称为发布-订阅（Publish/Subscribe）模式，属于行为型模式的一种，它定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态变化时，会通知所有的观察者对象，使他们能够自动更新自己



这里以微信公众号及订阅用户为例：

**抽象观察者（Observer):**

```java
// 定义一个更新方法
public interface Observer {
    public void update(String message);
}
```

**具体观察者实现类（可以是多个不同的类）：**

```java 
// 微信用户是观察者，实现更新方法
public class WeixinUser implements Observer {
    // 微信用户名
    private String name;
    public WeixinUser(String name) {
        this.name = name;
    }
    @Override
    public void update(String message) {
        System.out.println(name + "-" + message);
    }
}

```

##### **抽象被观察者（Subject）**

```java
//  抽象主题，有三个方法
public interface Subject {
    /**
     * 增加订阅者
     */
    public void attach(Observer observer);
    /**
     * 删除订阅者
     */
    public void detach(Observer observer);
    /**
     * 通知订阅者更新消息
     */
    public void notify(String message);
}
```

**具体被观察者：**

​		公众号是具体主题（具体被观察者），里面存储了订阅该公众号的微信用户，并实现了抽象主题中的方法：

```java
public class SubscriptionSubject implements Subject {
    //储存订阅公众号的微信用户
    private List<Observer> weixinUserlist = new ArrayList<Observer>();

    @Override
    public void attach(Observer observer) {
        weixinUserlist.add(observer);
    }

    @Override
    public void detach(Observer observer) {
        weixinUserlist.remove(observer);
    }

    @Override
    public void notify(String message) {
        for (Observer observer : weixinUserlist) {
            observer.update(message);
        }
    }
}
```

**客户端调用：**

```java
public class Client {
    public static void main(String[] args) {
        SubscriptionSubject mSubscriptionSubject=new SubscriptionSubject();
        //创建微信用户
        WeixinUser user1=new WeixinUser("杨影枫");
        WeixinUser user2=new WeixinUser("月眉儿");
        WeixinUser user3=new WeixinUser("紫轩");
        //订阅公众号
        mSubscriptionSubject.attach(user1);
        mSubscriptionSubject.attach(user2);
        mSubscriptionSubject.attach(user3);
        //公众号更新发出消息给订阅的微信用户
        mSubscriptionSubject.notify("刘望舒的专栏更新了");
    }
}
```

