**定义：**

​		通常在编码时为了扩展一个类的功能使用继承来实现，但是继承的缺点主要是单继承的局限性和可能产生类爆炸的后果。

​		装饰模式（Decorator）,动态地给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生成子类更为灵活。装饰模式属于结构型模式，它是作为现有的类的一个包装。

**优点：**

- 装饰类和被装饰类可以独立发展，不会相互耦合。
- 装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能。

**缺点：**

- 多层装饰可能比较复杂

**使用场合：**

- 如果你希望在无需修改代码的情况下即可使用对象， 且希望在运行时为对象新增额外的行为， 可以使用装饰模式。
- 如果用继承来扩展对象行为的方案难以实现或者根本不可行， 你可以使用该模式。

```java
public interface Troll { // 巨魔接口

  void attack();
  int getAttackPower();
  void fleeBattle();

}
```

```java
// 简单巨魔实现
public class SimpleTroll implements Troll {

  @Override
  public void attack() {
    LOGGER.info("巨魔尝试抓住你!");
  }

  @Override
  public int getAttackPower() {
    return 10;
  }

  @Override
  public void fleeBattle() {
    LOGGER.info("巨魔尖叫着逃跑");
  }
}

```

```java
/**
 * Decorator 为巨魔添加一个木棍.
 */
@Slf4j
@RequiredArgsConstructor
public class ClubbedTroll implements Troll {

  private final Troll decorated;

  @Override
  public void attack() {
    decorated.attack();
    LOGGER.info("巨魔挥舞着木棍想你冲来");
  }

  @Override
  public int getAttackPower() {
    return decorated.getAttackPower() + 10;
  }

  @Override
  public void fleeBattle() {
    decorated.fleeBattle();
  }
}
```

```java 
public static void main(String[] args) {

    // simple troll
    LOGGER.info("一个简单的巨魔方法.");
    var troll = new SimpleTroll();
    troll.attack();
    troll.fleeBattle();
    LOGGER.info("巨魔力量: {}.\n", troll.getAttackPower());

    // 通过装饰器为巨魔添加其他行为
    LOGGER.info("一个拥有巨大棍棒的巨魔会让你大吃一惊.");
    var clubbedTroll = new ClubbedTroll(troll);
    clubbedTroll.attack();
    clubbedTroll.fleeBattle();
    LOGGER.info("Clubbed troll power: {}.\n", clubbedTroll.getAttackPower());
  }
```

