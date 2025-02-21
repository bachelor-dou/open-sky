**定义：**

​		使多个对象都有机会处理请求，从而避免了请求的发送者和接受者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处理它为止。将所有处理者形成一条链，在链中决定哪个对象能够处理请求，并返回结果，不能处理则继续向下传递请求。

**优点：**

- 将请求和处理分开，请求者不需要知道是谁处理的，处理者可以不用知道请求的全貌。
- 所有具体处理者都进行了解耦，互相之间毫无关系。如果后续如果有新的处理逻辑需要加入，只需要继承抽象处理者，实现相关的处理逻辑，具体处理者就可以进行工作了，无需修改其他任何代码

**缺点：**

- 性能问题，每个请求从链头遍历到链尾，如果链比较长则性能低下。
- 调试问题，属于递归调用，调试不方便。

### 例1

#### 处理抽象类

​		所有的处理者都是独立的，并且每个处理者应该都具备相同的行为----处理逻辑

```java
public interface RequestHandler {

  boolean canHandleRequest(Request req);

  int getPriority();

  void handle(Request req);

  String name();

```

#### 具体处理类

```java
@Slf4j
public class OrcOfficer implements RequestHandler {
  @Override
  public boolean canHandleRequest(Request req) {
    return req.getRequestType() == RequestType.TORTURE_PRISONER;
  }

  @Override
  public int getPriority() {
    return 3;
  }

  @Override
  public void handle(Request req) {
    req.markHandled();
    LOGGER.info("{} handling request \"{}\"", name(), req);
  }

  @Override
  public String name() {
    return "Orc officer";
  }
}
```

```java
@Slf4j
public class OrcSoldier implements RequestHandler {
  @Override
  public boolean canHandleRequest(Request req) {
    return req.getRequestType() == RequestType.COLLECT_TAX;
  }

  @Override
  public int getPriority() {
    return 1;
  }

  @Override
  public void handle(Request req) {
    req.markHandled();
    LOGGER.info("{} handling request \"{}\"", name(), req);
  }

  @Override
  public String name() {
    return "Orc soldier";
  }
}
```

```java
@Slf4j
public class OrcCommander implements RequestHandler {
  @Override
  public boolean canHandleRequest(Request req) {
    return req.getRequestType() == RequestType.DEFEND_CASTLE;
  }

  @Override
  public int getPriority() {
    return 2;
  }

  @Override
  public void handle(Request req) {
    req.markHandled();
    LOGGER.info("{} handling request \"{}\"", name(), req);
  }

  @Override
  public String name() {
    return "Orc commander";
  }
}
```



#### 处理链路封装类：

```java
public class OrcKing {

  private List<RequestHandler> handlers;

  public OrcKing() {
    buildChain();
  }

  private void buildChain() {
    handlers = Arrays.asList(new OrcCommander(), new OrcOfficer(), new OrcSoldier());
  }

  /**
   * Handle request by the chain.
   */
  public void makeRequest(Request req) {
    handlers
        .stream()
        .sorted(Comparator.comparing(RequestHandler::getPriority))
        .filter(handler -> handler.canHandleRequest(req))
        .findFirst()
        .ifPresent(handler -> handler.handle(req));
  }
}
```



#### AppTest 类

```java
public static void main(String[] args) {
    // var king = new OrcKing();
    OrcKing king = new Orcing();
    king.makeRequest(new Request(RequestType.DEFEND_CASTLE, "defend castle"));
    king.makeRequest(new Request(RequestType.TORTURE_PRISONER, "torture prisoner"));
    king.makeRequest(new Request(RequestType.COLLECT_TAX, "collect tax"));
  }
```



#### 请求类封装

```java
// 请求类封装
public class Request {

  private final RequestType requestType;

  private final String requestDescription;

  private boolean handled;  // 请求是否被处理

  public Request(final RequestType requestType, final String requestDescription) {
    this.requestType = Objects.requireNonNull(requestType);
    this.requestDescription = Objects.requireNonNull(requestDescription);
  }

  public String getRequestDescription() {
    return requestDescription;
  }

  public RequestType getRequestType() {
    return requestType;
  }

  /**
   * Mark the request as handled.
   */
  public void markHandled() {
    this.handled = true;
  }

  public boolean isHandled() { // 是否被处理
    return this.handled;
  }

  @Override
  public String toString() {
    return getRequestDescription();
  }
```



```
// 三种请求类型枚举类
public enum RequestType {

  DEFEND_CASTLE,
  TORTURE_PRISONER,
  COLLECT_TAX
}
```





### 例2

场景： 依据过期Redis key的字符串开头不同，分给不同的处理器进行处理。

#### **处理抽象类**

```java
public abstract class AbstractRedisExpireHandle {
    //key的前缀，用于区分是什么业务
    private String prefix;

    //构造，子类必须实现
    public AbstractRedisExpireHandle(){}

    /**
     * 操作前缀属性的公开方法
     * @return
     */
    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    //抽象的，所有具体处理者应该实现的处理逻辑
    abstract void expireHandle(String redisKey);
}
```

#### **具体处理类（多个）**

```java
/**
 * 红包过期
 */
@Component
public class RedPacketExpireHandle extends AbstractRedisExpireHandle {
 
    //设置业务前缀
    private static final String PREFIX = "kb";
 
    @Reference(version = "1.0.0")
    private TaskService taskService;
 
    public RedPacketExpireHandle(){
        setPrefix(PREFIX);
    }
 
    //实现具体处理逻辑
    @Override
    void expireHandle(String redisKey) {
        taskService.redPacketExpireHandle(redisKey);
    }
}
```





#### **处理类链路封装：**

```java
@Component
public class ExecuteHandle implements ApplicationContextAware,InitializingBean {
 
    //spring容器
    private ApplicationContext context;
 
    //具体处理者的集合
    private List<AbstractRedisExpireHandle> expireHandles = new ArrayList<>();
 
    //该方法会在容器启动的时候被调用
    @Override
    public void afterPropertiesSet() throws Exception {
        //从容器中找到所有继承了抽象处理者的类，并加入到集合中，从而形成处理链
        String[] beanNames =  BeanFactoryUtils.beanNamesForTypeIncludingAncestors(context,AbstractRedisExpireHandle.class);
        for(int i=0;i<beanNames.length;i++){
            expireHandles.add((AbstractRedisExpireHandle)context.getBean(beanNames[i]));
        }
    }
 
    //该方法会将spring的当前容器传递进来
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.context = applicationContext;
    }
 
 
    //遍历处理链，通过前缀来判断，能否处理逻辑，如果不能则继续遍历
    public void handle(String redisKey){
        if(expireHandles.size() > 0){
            for(AbstractRedisExpireHandle abstractRedisExpireHandle : expireHandles){
                if(redisKey.startsWith(abstractRedisExpireHandle.getPrefix())){
                    abstractRedisExpireHandle.expireHandle(redisKey);
                }
            }
        }
    }
}
```

 

#### **appTest类：**

```java
@Component
public class RedisExpireListener implements MessageListener {
 
    @Autowired
    private ExecuteHandle executeHandle;
 
    @Override
    public void onMessage(Message message, byte[] bytes) {
        byte[] body = message.getBody();
        String redisKey = new String(body);
        executeHandle.handle(redisKey);
    }
}
```

