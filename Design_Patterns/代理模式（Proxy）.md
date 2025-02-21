**使用场景：**

​	即原有的业务复杂或者庞大，难以修改，而现在需要扩展一些功能，这里就需要代理模式实现。

​	有静态代理和动态代理两种模式：

​		静态代理服务于某个接口实现类，即一个代理只能服务于一个特定的业务实现类。

​		**而动态代理可以代理某个业务接口，需要那个实现类再传入哪个。**



**主要有四种用途：**

​		**远程代理，**这种方式通常是为了隐藏目标对象存在于不同地址空间的事实，方便客户端访问。例如，用户申请某些网盘空间时，会在用户的文件系统中建立一个虚拟的硬盘，用户访问虚拟硬盘时实际访问的是网盘空间。

​		**虚拟代理**，这种方式通常用于要创建的目标对象开销很大时。例如，下载一幅很大的图像需要很长时间，因某种计算比较复杂而短时间无法完成，这时可以先用小比例的虚拟代理替换真实的对象，消除用户对服务器慢的感觉。

​		**保护代理，**当对目标对象访问需要某种权限时，保护代理提供对目标对象的受控保护，例如，它可以拒绝服务权限不够的客户。

​		**智能指引，**主要用于调用目标对象时，代理附加一些额外的处理功能。例如，增加计算真实对象的引用次数的功能，这样当该对象没有被引用时，就可以自动释放它(C++智能指针)；例如上面的房产中介代理就是一种智能指引代理，代理附加了一些额外的功能，例如带看房等。

**优点：**

- 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用；
- 代理对象可以扩展目标对象的功能；
- 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度，增加了程序的可扩展性

**缺点：**

- 静态代理模式会造成系统设计中类的数量增加，但动态代理可以解决这个问题；
- 在客户端和目标对象之间增加一个代理对象，会造成请求处理速度变慢；
- 增加了系统的复杂度；

**1. 首先是一个巫师塔接口及其实现类**

```java
public interface WizardTower {

  void enter(Wizard wizard);
}

@Slf4j
public class IvoryTower implements WizardTower {

  public void enter(Wizard wizard) {
    LOGGER.info("{} enters the tower.", wizard);
  }

}
```

**2. 巫师类**

```java
public class Wizard {
  private final String name;

  public Wizard(String name) {
    this.name = name;
  }

  @Override
  public String toString() {
    return name;
  }
}
```

**3.  巫师塔的代理类**

```java
@Slf4j   // 控制进入巫师的数量
public class WizardTowerProxy implements WizardTower {

  private static final int NUM_WIZARDS_ALLOWED = 3;

  private int numWizards;

  private final WizardTower tower;

  public WizardTowerProxy(WizardTower tower) {
    this.tower = tower;
  }

  @Override
  public void enter(Wizard wizard) {
    if (numWizards < NUM_WIZARDS_ALLOWED) {
      tower.enter(wizard);
      numWizards++;
    } else {
      LOGGER.info("{} is not allowed to enter!", wizard);
    }
  }
}
```

**4.  appTest类**

```java
var proxy = new WizardTowerProxy(new IvoryTower());
proxy.enter(new Wizard("Red wizard"));
proxy.enter(new Wizard("White wizard"));
proxy.enter(new Wizard("Black wizard"));
proxy.enter(new Wizard("Green wizard"));
proxy.enter(new Wizard("Brown wizard"));

Output：
Red wizard enters the tower.
White wizard enters the tower.
Black wizard enters the tower.
Green wizard is not allowed to enter!
Brown wizard is not allowed to enter!
```

### 万能动态代理（多个接口）：

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

//业务接口
interface DateService {
    void add();
    void del();
}

interface OperateService {
    void plus();
    void subtract();
}
//  两个接口分别的实现类
class DateServiceImplA implements DateService {
    @Override
    public void add() {
        System.out.println("成功添加！");
    }

    @Override
    public void del() {
        System.out.println("成功删除！");
    }
}

class OperateServiceImplA implements OperateService {
    @Override
    public void plus() {
        System.out.println("+ 操作");
    }

    @Override
    public void subtract() {
        System.out.println("- 操作");
    }
}

//万能的代理模板，代理多个接口
class ProxyInvocationHandler implements InvocationHandler {
    private Object service;

    public ProxyInvocationHandler(Object service) {
        this.service = service;
    }

    public Object getDateServiceProxy() {
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), service.getClass().getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        var result = method.invoke(service, args); // 方法返回值
        System.out.println(proxy.getClass().getName() + "代理类执行" + method.getName() + "方法，返回" + result +  "，记录日志！");
        return result;
    }
}


//客户端
public class Test {
    public static void main(String[] args) {
        DateService dateServiceA = new DateServiceImplA();
        DateService dateServiceProxy = (DateService) new ProxyInvocationHandler(dateServiceA).getDateServiceProxy();
        dateServiceProxy.add();
        dateServiceProxy.del();

        OperateService operateServiceA = new OperateServiceImplA();
        OperateService operateServiceProxy = (OperateService) new ProxyInvocationHandler(operateServiceA).getDateServiceProxy();
        operateServiceProxy.plus();
        operateServiceProxy.subtract();
    }
}
/*
成功添加！
$Proxy0代理类执行add方法，返回null，记录日志！
成功删除！
$Proxy0代理类执行del方法，返回null，记录日志！
+ 操作
$Proxy1代理类执行plus方法，返回null，记录日志！
- 操作
$Proxy1代理类执行subtract方法，返回null，记录日志！
*/

```

