**使用场景：**

​		模板模式(Template Pattern)，在一个抽象类公开定义了执行它的方法的模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。

​		模板方法模式，定义一个操作中的算法的骨架，而将一些步骤延迟到子类中，使得子类可以不改变一个算法的结构，就可以重定义该算法的某些特定步骤，这种类型的设计模式属于行为型模式。



**场景1：磨不同口味豆浆**

​	流程：选材—>添加配料—>浸泡—>放到豆浆机打碎，这里只有**添加配料**这一步骤，才是不同口味豆浆里的不同点；

```java
// 抽象类，表示豆浆	SoyaMilk.java
public abstract class SoyaMilk {
	// 模板方法：可以做成final，不让子类去覆盖
	final void make() {
		select();
		addCondiment();
        // if(customerWantCondiment()) addCondiment(); // 钩子方法	
		soak();
		beat();
	}
	//钩子方法：给子类赋予是否增加配料的决定权
	// boolean customerWantCondiment() return true;// 默认情况添加加配料
    
	//选材料
	void select() { System.out.println("第一步：选择新鲜的豆子"); }
	//添加不同的配料：抽象方法，由子类具体实现
	abstract void addCondiment();
	//浸泡
	void soak() { System.out.println("第三步：豆子和配料开始浸泡3H"); }
	//榨汁
	void beat() { System.out.println("第四步：豆子和配料放入豆浆机榨汁"); }
}


// RedBeanSoyaMilk.java
public class ReadBeanSoyaMilk extends SoyaMilk {
	@Override
	void addCondiment() {
		System.out.println("第二步：加入上好的红豆");
	}
}

// PeanutSoyMilk.java
public class PeanutSoyaMilk extends SoyaMilk {
	@Override
	void addCondiment() {
		System.out.println("第二步：加入上好的花生");
	}
}

// Client.java
public class Client {
	public static void main(String[] args) {
		System.out.println("=======制作红豆豆浆=======");
		SoyaMilk redBeanSoyaMilk = new ReadBeanSoyaMilk();
		redBeanSoyaMilk.make();
		
		System.out.println("=======制作花生豆浆=======");
		SoyaMilk peanutSoyaMilk = new PeanutSoyaMilk();
		peanutSoyaMilk.make();
	}
}


// 增加钩子方法后后
public class PureSoyaMilk extends SoyaMilk {
	@Override
	void addCondiment() {
		// 添加配料的方法 空实现 即可
	}
	@Override
	boolean customerWantCondiment() {
		return false;
	}
}

// Client.java
public class Client {
	public static void main(String[] args) {
		System.out.println("=制作纯豆浆=");
		SoyaMilk pureSoyMilk = new PureSoyaMilk();
		pureSoyMilk.make();
	}
```

**场景2：小偷偷东西**

流程：选择目标 —> 迷惑他视线 —> 偷东西，

​		首先介绍模板方法类及其具体实现。为了确保子类不会覆盖模板方法，模板方法（ steal）应该被声明为 final，否则在基类中定义的骨架可能会在子类中被覆盖。

```java
public abstract class StealingMethod {

  protected abstract String pickTarget();

  protected abstract void confuseTarget(String target);

  protected abstract void stealTheItem(String target);

  //  模版方法
  public final void steal() {  
    var target = pickTarget();
    LOGGER.info("The target has been chosen as {}.", target);
    confuseTarget(target);
    stealTheItem(target);
  }
}


@Slf4j
public class SubtleMethod extends StealingMethod {

  @Override
  protected String pickTarget() {
    return "shop keeper";
  }

  @Override
  protected void confuseTarget(String target) {
    LOGGER.info("Approach the {} with tears running and hug him!", target);
  }

  @Override
  protected void stealTheItem(String target) {
    LOGGER.info("While in close contact grab the {}'s wallet.", target);
  }
}

@Slf4j
public class HitAndRunMethod extends StealingMethod {

  @Override
  protected String pickTarget() {
    return "old goblin woman";
  }

  @Override
  protected void confuseTarget(String target) {
    LOGGER.info("Approach the {} from behind.", target);
  }

  @Override
  protected void stealTheItem(String target) {
    LOGGER.info("Grab the handbag and run away fast!");
  }
} 
```

//  包含模版方法的盗贼类

```java
public class HalflingThief {

  private StealingMethod method;

  public HalflingThief(StealingMethod method) {
    this.method = method;
  }

  public void steal() {
    method.steal();
  }

  public void changeMethod(StealingMethod method) {
    this.method = method;
  }
}
```

App类：

```java
var thief = new HalflingThief(new HitAndRunMethod());
    thief.steal();
    thief.changeMethod(new SubtleMethod());
    thief.steal();
```