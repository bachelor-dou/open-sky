**适配器模式让两个逻辑上不协调的模块可以协同工作**

场景： 

​	一个船长只会滑皮划艇航行，但现在只有一艘渔船，因此需要船长滑渔船离开

皮划艇接口 以及一艘渔船：

```java
// 皮划艇接口
public interface RowingBoat {

  void row();
}

// 渔船类
final class FishingBoat {

  void sail() {
    LOGGER.info("The fishing boat is sailing");
  }

    
// 船长类 客户端
public final class Captain {
  private RowingBoat rowingBoat;

  void row() {
    rowingBoat.row();
  }
}
    
    
// 适配器类
public class FishingBoatAdapter implements RowingBoat {
  private final FishingBoat boat = new FishingBoat();

  public final void row() {
    boat.sail();
  }

//  转换类
// The captain can only operate rowing boats but with adapter he is able to use fishing boats as well
public final class App {
  public static void main(final String[] args) {
    Captain captain = new Captain(new FishingBoatAdapter());
    captain.row();
  }
}

```

