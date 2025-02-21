

### JVM调优：

**JVM调优是为了调优GC：主要有两个点：**

- ==一个是减少Full gc的次数==
- ==一个是缩短一次Full gc的时间，Full GC时会发生==。

​		**GC均会触发STW（暂停（挂起）一切线程操作），但是minorGC时间更短，而Full GC时间很长，所以要尽量避免FullGC**（==将新对象预留在新生代，由于 Full GC 的成本远高于 Minor GC，因此尽可能将对象分配在新生代是明智的做法，实际项目中根据 GC 日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况，=。一般可通过==-Xmn将年轻代设置大点，这样时间较长，minorGC会触发的更多一些==）。

​		另外，还可以通过 **`-XX:NewRatio=1`** 来设置老年代与新生代内存的比值（默认是2）

#### Minor GC ，Major GC与 Full GC

​		  **1. Minor GC 的触发机制**：Minor GC 的触发是被动的，当程序不断的创建新对象，JVM 会往 Eden 区塞，**当 Eden 区内存空间满了的时候，就会触发 Minor GC**，需要注意的是，**Survivor 0 满不会触发 Minor GC**。那么**当eden区满了，而此时Survivor 0也满，放不下时，此时==会对S0区与eden区同时进行GC Roots可达性分析==，将所有存活的对象复制到S1区并且将S0区与eden区清空 ( 这里如果复制的对象大小超过S1区50%的容量会直接放入老年区 ) ，并且交换0区与1区**

​		**2.Major GC** 又称 Old GC，因为其主要收集的内存区域是老年代，老年代的空间一般比新生代要大，这里发生的 GC 次数理当且应当比较少


​		 **3.  Full GC，覆盖了整个堆空间，包括元空间**， FULL GC的 STW 时间会更长



**-Xms  堆最小 值** 

**-Xmx  堆大小最大**

**-Xmn 年轻代大小**

**-Xss  线程栈大小：一般不用调整**

**-XX:NewRatio   默认为2 。调整的话表示为 -XX:NewRatio=4。**

**-XX:SurvivorRatio  默认为8 ， 表示 两个Survivor ： Eden = 2 ： 8 。**



-XX:+UseAdaptiveSizePolicy     （JDK8默认使用的 -XX:+UseParallelGC垃圾收集器。这个垃圾收集器默认开启这个功能）

-XX:-UseAdaptiveSizePolicy（可以手动关闭）

> ​		AdaptiveSizePolicy(自适应大小策略) 是 JVM GC Ergonomics(自适应调节策略) 的一部分。
>
> ​		如果开启 AdaptiveSizePolicy，则每次 GC 后会重新计算 Eden、From 和 To 区的大小，计算依据是 GC 过程中统计的 **GC 时间、吞吐量、内存占用量**。
>
> ​		该策略会在设定的堆大小内**自动调整**e区和S区的大小，而不是按照-XX:SurvivorRatio ：8 来进行分配，**一般都会使幸存区比较小，导致minorGC后eden区存活对象放不下去S区进入老年代**，从而老年代更容易满，导致FullGC, STW时间更长



### 1. **Java开发中可能碰到的问题**：

- Oom内存不足（分为堆内存不足和非堆内存不足两种OOM）
- 内存泄漏
- 线程死锁
- 锁争用
- Java进程消耗CPU过高

​     上述问题一般重启服务或者调大内存可以解决。

 **前置条件： Java系统进程所使用的物理内存（Res）会大于设置的Xmx(JVM堆内存的值)。Java进程的内存分为堆内存以及非堆内存（元空间/方法区）：**

​		**堆内存**是供Java应用程序使用的；堆内存内部各组成的大小可以通过JVM的一系列命令行参数来控制。

 	   非堆内存则没有相应的参数来控制大小，默认状态下其大小就是系统内存的大小，会一直扩大，元空间内存不够用的情况下就会报OOM，其依赖于进程内存的最大值以及生成的Java字节码大小、创建的线程数量、维持java对象的状态信息大小（用于GC）以及一些第三方的包。在java中每new一个线程，jvm都是向操作系统请求new一个本地线程，此时操作系统会使用剩余的内存空间来为线程分配内存，而不是使用jvm的内存。

​		元空间大小默认值为20.8M，随着类加载越来越多不断扩容调整，上限是`-XX:MaxMetaspaceSize`，默认是几乎无穷大，可进行设置（`-XX:MetaspaceSize`和`-XX:MaxMetaspaceSize`）两个参数如何设置

![image-20221021120433289](pics\image-20221021120433289.png)

   

**175跟176 环境上目前camunda的jvm参数为默认设置，即堆内存8G（太大需优化，下图为176的**

​               ![image-20221020200822239](pics\image-20221020200822239.png)

### 2. Java进程监控及性能调优工具

#### 2.1  CPU过高分析

​        使用top命令查看CPU、内存使用状态时，在Linux下java进程占用CPU高的分析手段主要是使用Linux命令查出高CPU的进程，分析其是由于进程原因还是系统原因，确定为进程原因后，使用jstack工具进行分析。

-    **jstack命令**

  1.  top命令确定占用率较高的进程pid后。

  2.  top -Hp pid   可以查看该进程下所有线程的CPU使用情况，确定使用CPU较高的线程pid

  3.  使用jstack命令对2步骤得到的线程pid进行堆栈状态查看。**jstack  pid 命令可以理解为生成当前jvm线程的快照**。jstack命令生成的  thread  dump  信息包含了JVM中所有存活的线程，每一个线程都有一个对应的nid。为了分析指定线程，必须找出对应线程的调用栈，将线程pid转化为16进制值（命令：printf "%x\n" tid）后，通过**搜索命令在  thread  dump  中找对应的nid。 隔一段时间在执行一次，区分两次dump（快照）是否有区别，就能够定位到对应的代码行**

      **jstack -l [PID] >> /tmp/log.txt**：表示可以将这些堆栈信息打到一个文件里，后续进行分析
  
  ![image-20221020142256100](pics\image-20221020142256100.png)

​       以上例子，**对应的nid=0x246c调用栈中发现该线程一直执行JstackCase类中第33行的calculate方法，因此可检查此处的代码是否有问题。**

   此外还可通过**thread  dump  分析各个线程当前的运行情况： 比如是否存在死锁、是否存在一个线程长时间持有锁不放等。  这其中线程一般处于以下几种状态：RUNNABLE（线程处于执行中）；BLOCKED(线程处于阻塞状态) ；WAITING(线程正在等待实例：多线程竞争锁)**

![image-20221020144605231](pics\image-20221020144605231.png)

​        以上例子中：

​            线程1获取到了锁，在RUNNABLE状态下，线程2为BLOCK状态。其中locked<0x....2208>说明线程1对地址为0x....2208的对象进行了加锁；

​            而前一个线程中waiting  to lock<0x000.......2208>说明线程2正在等待线程1中<0x....2208>对象上的锁。  

​            waiting  for  monitor ........  [0x000000001e21f000]  说明线程1是通过synchronized关键字进入了监视器的临界区，并处于"Entry Set"队列，等待monitor

#### 2.2  内存过高分析

- 使用   **pmap  进程id**   查看进程的内存   (用/proc/{pid}/smaps可以进行更详细查看)

​	第一列。内存块起始地址

​	第二列。占用内存大小

​	第三列，内存权限

​	第四列。内存名称。anon表示动态分配的内存，stack表示栈内存

​	最后一行。占用内存总大小，请注意，此处为虚拟内存大小，占用的物理内存大小能够通过top查看

- **使用 jmap -histo 进程id  查看堆内存使用状况，一般结合jhat使用**

  第一列，序号。

  第二列，对象实例数量

  第三列，对象实例占用总内存数。单位：字节

  第四列，对象实例名称

  最后一行，总实例数量与总内存占用数

  **jmap -heap 进程id**   查看当前进程设置的jvm参数。

  <img src="pics\image-20221020201041694.png" alt="image-20221020201041694" style="zoom:67%;" />

- **使用jstat -gc  进程id  5000  (该命令表示：每5秒显示一次该java进程的GC情况。。5000表示5秒，)**

​       jstat 命令可以查看Java内存分布及回收情况

​		**jstat -gcnew pid** 新生代gc查看

​		**jstat -gcold**          老年代gc 查看

### 3.设置java运行内存以及CPU核心数

#### 3.1 设置java运行内存（jvm设置）

```
java -jar -Xms128M -Xmx256m -xx:PermSize=128m -xx:MaxPermSize=256m xxxx.jar &
```

**-Xss：**规定了每个线程虚拟机栈的大小（影响并发线程数大小）

**-Xms128M** 配置初始堆内存128m(超过初始值会扩容到最大值)

**-Xmx256m等价于-XX：MaxHeapSize** 配置最大堆内存256m(通常要将初始值和最大值设置为一样，因为扩容会导致内存抖动，影响程序运行稳定性)

**-XX:MaxNewSize**=512m 新生代内存上限值

**&：** 加到一个命令的最后面，表示这个命令放在后台执行

（JVM堆内存是指java程序运行时可以调配使用的内存空间。在JVM启动时会自动设置，一般为物理内存的1/64，最大为1/4。在JVM98%用于GC时，而可用内存不足2%时，就会发生OOM）

**一般要设置-Xmx跟-Xms的大小一致，这样GC的运行效率会高一些，因为不再需要估算堆是否需要调整大小了**

   以下为全部-X参数的设置

<img src="pics\image-20221020165315210.png" alt="image-20221020165315210" style="zoom: 67%;" />

-XX  参数（**主要用于JVM调优和debug**）

​       -XX:[+/-]<name> = <value>  表示name的属性值为value。

​      例子：-XX:MaxGCPauseMillis=500(表示设置GC的最大停顿时间是500ms)

#### 3.2 设置指定CPU核心来运行进程

- taskset命令

  **tasket可以具体查看某一进程（或线程）运行在哪个CPU上，也可以使某个程序运行在某个或某些CPU上**

  taskset -p 0-1 进程ID（设置指定索引为(0\1)核处理）

  taskset -p 2 进程ID（设置指定索引为(2)的核处理）

​       

### 3.  远程debug服务

#### VisualVM 分析JVM堆工具

​     使用VisualVM可以远程监控Linux服务器下的JVM

[(22条消息) 使用Java VisualVM监控远程JVM（远程服务器为linux配置）_紫漪的博客-CSDN博客_java visualvm使用](https://blog.csdn.net/u011391839/article/details/76984995)

此外也可使用jstack来进行远程排查堆栈问题，定位代码问题点

### 4. 线上系统响应慢的问题

​		系统响应慢，可能原因为CPU%及FullGC次数过多的问题。线上系统出现这个问题进行排查时。首先要做的是导出jstack和内存信息，然后重启系统，尽快保证可用性。这种情况的原因主要有两种：[教你如何排查Java系统运行缓慢等问题 附详细思路 | w3c笔记 (w3cschool.cn)](https://appapi.w3cschool.cn/article/66414768.html)

1. 代码中某个位置读取数据量大，导致系统的内存耗尽。从而导致Full GC次数过多，Full GC时进程会暂停一切服务，因此系统相应会很慢；

2. 代码中有比较消耗CPU的操作。导致CPU过高，系统运行缓慢；

   **以上两种线上出的频率都比较高，此外还有一些次要原因导致运行较慢：**

3. 代码在某个位置有阻塞性操作，导致该功能调用比较耗时，但该问题点出现是比较随机的

4. 某个线程由于某种原因进入了WAITING状态，此时该功能整体不可用，

5. 锁的使用不当导致多个线程进入了死锁状态，从而系统运行整体缓慢。

  **上述这三种情况，可以通过查看系统日志来一步步甄别。**

- **Full GC次数太多**

  主要特征：多个线程CPU都超过100%，通过jstack命令可以看到这些线程主要是垃圾回收线程，通过jstat命令监控GC情况，可以看到Full GC 次数（非常多），可能还在不断增加。（这种情况下，就会得到VM Thread之类的线程，然后可通过jstat  -gcutil pid命令来监控当前系统的GC情况）

- **比较耗时的计算导致的CPU过高**

   top——top -Hp 进程id ——  jstack  线程id (查看原因，如果是代码中比较耗时的计算，得到的就是一个线程的具体堆栈信息，而如果是Full GC次数过多，就会是VM Thread)

- **不定期出现的接口耗时现象**

  这种情况就是某个接口访问需要2-3s才能返回（这种情况一般的CPU消耗和内存占用并不高）。上述jstack的方法就无法判断了。

  **定位思路应该是：** **找到该接口，通过压测工具不断加大访问力度，如果该接口中某个位置比较耗时，那么在访问频率高的情况下，大多数线程都将阻塞于该位置处。这时候查看多个线程的堆栈日志，就能定位到该阻塞点。**



**此外其他排查接口响应慢的排查方法以及解决方案：**

​	**法1：** 

- SkyWalking 是分布式系统的应用程序性能监视工具（又叫链路追踪工具，会展示出一个与网络有关的耗时），可用于排查接口响应慢的问题。比如读写数据库，读写redis,spingCloud调用等。 也可使用
- httpstat:  一个检查网站性能的curl统计分析工具

​	**法2：** 看代码猜测问题点 。响应慢一般就是操作数据库耗时比较长。

​	**法3：**在相应的链路上打印日志，然后查看日志（不推荐）

​	**数据库耗时长：**

-  必要字段加索引
- 确实索引是否失效
- 有回表查询，尽量优化为覆盖索引

**架构不合理**

**业务逻辑**      写代码的思路清晰，

**java死锁**   jps+jstack、jconsole、jvisualvm三种方式



### 元空间  

​		[深入理解堆外内存 Metaspace](https://javadoop.com/post/metaspace)

​		[Metaspace GC 问题排查](https://blog.csdn.net/dkangel/article/details/121276741)

[		为什么设置-Xmx4g但是java进程内存占用达到8g？](https://blog.csdn.net/w1014074794/article/details/113340344)

​		Metaspace区域位于堆外，因此最大内存大小取决于系统内存，而不是堆大小。 Metaspace 是用来存放 class metadata 的，class metadata 用于记录一个 Java 类在 JVM 中的信息，包括但不限于JVM运行时数据： 

1、Klass 结构，这个非常重要，把它理解为一个 Java 类在虚拟机内部的表示吧；

2、method metadata，包括方法的字节码、局部变量表、异常表、参数信息等；

3、常量池；

4、注解；

5、方法计数器，记录方法被执行的次数，用来辅助 JIT 决策

​		==当一个类被加载时，它的类加载器会负责在 Metaspace 中分配空间用于存放这个类的元数据。而分配给一个类的空间，是归属于这个类的类加载器的，只有当这个类加载器卸载的时候，这个Metaspace空间才会被释放。释放 Metaspace 的空间，并不意味着将这部分空间还给系统内存，这部分空间通常会被 JVM 保留下来。这部分被保留的空间有多大，取决于 Metaspace 的碎片化程度。==



​		Java 8以后，关于元空间的JVM参数有两个：`-XX:MetaspaceSize=N`和 `-XX:MaxMetaspaceSize=N`，对于64位JVM来说，**元空间的默认初始大小是20.75MB，默认的元空间的最大值是无限**（MetaspaceSize的默认大小取决于平台，范围从12 MB到大约20 MB）。

​		MetaspaceSize表示metaspace首次使用不够而触发Full GC的阈值，只对触发起作用。

​		MaxMetaspaceSize用于设置metaspace区域的最大值，这个值可以通过mxbean中的MemoryPoolBean获取到，如果这个参数没有设置，那么就是通过mxbean拿到的最大值是-1，表示无穷大。

​		由于调整元空间的大小需要Full GC，这是非常昂贵的操作，如果应用在启动的时候发生大量Full GC，通常都是由于永久代或元空间发生了大小调整，基于这种情况，一般建议在JVM参数中将MetaspaceSize和MaxMetaspaceSize设置成一样的值，并设置得比初始值要大。



#### **Metaspace 可能在两种情况下触发 GC：**

1、分配空间时：虚拟机维护了一个阈值，如果 Metaspace 的空间大小超过了这个阈值，那么在新的空间分配申请时，虚拟机首先会通过收集可以卸载的类加载器来达到复用空间的目的，而不是扩大 Metaspace 的空间，这个时候会触发 GC。这个阈值会上下调整，和 Metaspace 已经占用的操作系统内存保持一个距离。

2、碰到 Metaspace OOM：Metaspace 的总使用空间达到了 MaxMetaspaceSize 设置的阈值，或者 Compressed Class Space 被使用光了，如果这次 GC 真的通过卸载类加载器腾出了很多的空间，这很好，否则的话，会进入一个糟糕的 GC 周期，即使我们有足够的堆内存。



#### MaxMetaspaceSize会耗尽内存吗？

​		参数**MaxMetaspaceFreeRatio**将元空间限制从其有效的无限默认值减少，将具有完全不同的目的：避免无限的元空间增长。**实际上，一个名为`MaxMetaspaceFreeRatio`（默认为70％）的设置，这意味着实际的元空间大小永远不会超过其占用率的230％。**

​		使用`java -XX:+PrintFlagsInitial`查看本机参数后发现：会发现防止因为某些情况导致Metaspace无限的使用本地内存，影响到其他程序。**在本机上该MaxMetaspaceSize参数有默认值**。

​		==**因此，在实践中，除非应用程序不断泄漏类装入器/类或生成大量动态代码，否则实际的元空间大小应稳定在与其实际需要接近的范围内。**==（不无限增长，耗尽内存，不代表不会发生元空间OOM）

#### 结论

**MaxMetaspaceSize如果不指定大小的话，不会耗尽内存**，原因如下：

1. MetaspaceSize的扩容会Full GC， 回收后程序有更多可用空间，可能之后空间够用不会再触发扩容。
2. MaxMetaspaceFreeRatio控制高水位，要使其增长，首先必须将其填满，强制进行垃圾回收（元空间已满），以尝试释放对象，并且只有当它不能满足其`MinMetaspaceFreeRatio`（默认40％）目标时，才将当前元 空间扩展到不超过GC周期后占用率达到230％，垃圾收集之后，高水位线可能会升高或降低。
3. -XX:MaxMetaspaceSize=N 这个参数用于限制Metaspace增长的上限，防止因为某些情况导致Metaspace无限的使用本地内存，影响到其他程序。

### 直接内存

[JVM系列（九）直接内存（Direct Memory)](https://juejin.cn/post/6970670519720869919)

[直接内存 DirectMemory 详解](https://cloud.tencent.com/developer/article/1586341)

- 直接内存不是虚拟机运行时数据区的一部分，也不是《Java虚拟机规范》中定义的内存区域
- 直接内存是在Java堆外的、直接向系统申请的内存空间
- 来源于`NIO（New Input/Output）`类，通过存在堆中的`DirectByteBuffer`操作Native内存。它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆里面的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆中来回复制数据
- 通常，访问直接内存的速度会优于Java堆，即读写性能高。因此处于性能考虑，读写频繁的场合可能会考虑使用直接内存。Java的NIO库允许Java程序使用直接内存，用于数据缓冲区

