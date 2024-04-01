枫叶云链接：http://cloud.fynote.com/s/4976

# JVM面试题大全                Lecturer ：严镇涛

## 1.为什么需要JVM，不要JVM可以吗？

1.JVM可以帮助我们屏蔽底层的操作系统      一次编译，到处运行

2.JVM可以运行Class文件

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/a2b57e0612a0461dbd4ad1dfb42a6eca.png)

## 2.JDK，JRE以及JVM的关系

![03.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/ca7dac4d13c14d1191e3d860a1338d4d.png)

## 3.我们的编译器到底干了什么事？

仅仅是将我们的 .java 文件转换成了 .class 文件，实际上就是文件格式的转换，对等信息转换。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/bac9fb59888f45d0b5531cbc21ffecf9.png)

## 4.类加载机制

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1644577518000/b5ec599f1b8242a19cb6995a97cb02cf.png)

类加载机制其实就是虚拟机把Class文件加载到内存，并对数据进行校验，转换解析和初始化，形成可以虚拟机直接使用的Java类型，即java.lang.Class。

1.装载

Class文件  -- >二进制字节流 -->类加载器

1）通过一个类的全限定名获取这个类的二进制字节流

2）将这个字节流所代表的静态存储结构转换为方法区的运行时数据结构

3）在java堆中生成一个代表这个类的java.lang.Class对象，做为我们方法区的数据访问入口

2.链接：

1）验证：保证我们加载的类的正确性

* 文件格式验证
* 元数据验证
* 字节码验证
* 符号引用验证

2）准备

为类的静态变量分配内存，并将其初始化为当前类型的默认值。

private   static  int  a   = 1 ；    那么他在准备这个阶段  a = 0；

3）解析

*解析*是从运行时常量池中的符号引用动态确定具体值的过程。

把类中的符号引用转换成直接引用

3.初始化

执行到Clinit方法，为静态变量赋值，初始化静态代码块，初始化当前类的父类

## 5.类加载器的层次

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/5c442004ab9641c78585c238d88f57ea.png)

## 6.双亲委派机制

父类委托机制

源码       String              自己写  String

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/000d19eeb6c64783875c14592668f223.png)

## 7.如何打破双亲委派

1.复写

2.SPI   Service   Provider   Interface   服务提供接口    日志     Xml解析   JBDC

可拔插设计    可以随时替换实现

3.OSGI   热部署   热更新
![16461374670483013008ffy](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1/8acf0c26ba9146d89a17919f54f7925a.png)

## 8.运行时数据区

![20.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1644577518000/0b5a1e867e3240e6ba1809b074d54f01.png)

![16461374670483019699ffy](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1/6b3f162e4dbc46629ad360e8bb0f6994.png)1.方法区   线程共享 方法区是逻辑上堆的一部分 所以他有个名字：非堆

运行时常量池、字段和方法数据，以及方法和构造函数的代码，包括类和实例初始化和接口初始化中使用 的特殊方法

如果方法区域中的内存无法满足分配请求，Java 虚拟机将抛出一个 `OutOfMemoryError`

2.堆   线程共享   堆是为所有类实例和数组分配内存的运行时数据区域     内存不足 `OutOfMemoryError`

3.java虚拟机栈   执行java方法的        线程私有的      `StackOverflowError`

4.本地方法栈   执行本地方法      线程私有      `StackOverflowError`

5.程序计数器    记录程序执行到的位置    线程私有

## 9.栈帧结构是什么样子的？

![15.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1644577518000/9b75f075b66d48feba43ca971685731b.png)

附加信息：栈帧的高度，虚拟机版本信息

栈帧信息：附加信息+动态链接+方法的返回地址

局部变量表：方法中定义的局部变量以及方法的参数都会存放在这张表中  ，单纯的存储单元

操作数栈  以压栈以及出栈的方式存储操作数

举例：

int   a   =  1；

int   b   =  1；

int c = a + b ；

方法的返回地址：当你一个方法执行的时候，只有两种方式可以推出

1.遇到方法的返回的字节码指令

2.出现了异常，有异常处理器，则交给异常处理器  ，没有呢？抛异常

## 10.动态链接

动态链接是为了支持方法的动态调用过程 。

动态链接将这些符号方法引用转换为具体的方法引用

符号引用转变为直接引用      为了支持java的多态

void    a(){

b();

}

void    b(){

c();

}

void    c(){

}

## 11.java堆为什么要进行分代设计

![](file:///C:\Users\root\AppData\Local\Temp\ksohtml\wpsE144.tmp.jpg)![22.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/f9280f0ed7fb4dadb453c3004f4b2192.png)

新老年代划分

![16461374670483019381ffy](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1/91ccfbe811954b4eab7664b6792eaa51.png)

Eden区与S区

## 12.![](file:///C:\Users\root\AppData\Local\Temp\ksohtml\wps7F2D.tmp.jpg)老年代的担保机制

## 13.为什么Eden：S0：S1 是8：1：1![](file:///C:\Users\root\AppData\Local\Temp\ksohtml\wps942D.tmp.jpg)

98%的对象朝生夕死

## 14.对象的创建过程

![23.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1644577518000/0df155de2e8f429683e3faade45766e3.png)

## 15.方法区与元数据区以及持久代到底是什么关系

方法区    JVM规范

落地：JDK1.7之前   持久代     Perm Space   JVM虚拟机自己的内存

JDK1.8之后      元数据区 / 元空间     MetaSpace    直接内存

![16461374670483013249ffy](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1/d139d4d5c2764ad3ac6299a6a231ce40.png)

## 16.什么时候才会进行垃圾回收

> GC是由JVM自动完成的，根据JVM系统环境而定，所以时机是不确定的。
> 当然，我们可以手动进行垃圾回收，比如调用System.gc()方法通知JVM进行一次垃圾回收，但是
> 具体什么时刻运行也无法控制。也就是说System.gc()只是通知要回收，什么时候回收由JVM决
> 定。**但是不建议手动调用该方法，因为GC消耗的资源比较大**。

```
（1）当Eden区或者S区不够用了
（2）老年代空间不够用了
（3）方法区空间不够用了
（4）System.gc()     //通知      时机也不确定      执行的Full  GC
```

### 17. 如何确定一个对象是垃圾？

> **要想进行垃圾回收，得先知道什么样的对象是垃圾。**

* 引用计数法

**对于某个对象而言，只要应用程序中持有该对象的引用，就说明该对象不是垃圾，如果一个对象没有任何指针对其引用，它就是垃圾。**

`弊端`:如果AB相互持有引用，导致永远不能被回收。 循环引用    内存泄露   -->内存溢出

![16461374670483019208ffy](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1/206642b525724b559a09fa193940a322.png)

* 可达性分析/根搜索算法

**通过GC Root的对象，开始向下寻找，看某个对象是否可达**

![](file://E:/%E6%A1%8C%E9%9D%A2/yzt/%E7%AC%94%E8%AE%B0%E8%AF%BE%E4%BB%B6/JVM/3%E5%A4%A9JVM%E8%AE%AD%E7%BB%83%E8%90%A5/%E8%B5%84%E6%96%99+%E7%AC%94%E8%AE%B0/images/64.png?lastModify=1646659177)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/fe725834fb304a55a1bbae9aa70b7739.png)

> **能作为GC Root:类加载器、Thread、虚拟机栈的本地变量表、static成员、常量引用、本地方法栈的变量等。**

```
虚拟机栈（栈帧中的本地变量表）中引用的对象。
方法区中类静态属性引用的对象。
方法区中常量引用的对象。
本地方法栈中JNI（即一般说的Native方法）引用的对象。
```

## 18.对象被判定为不可达对象之后就“死”了吗

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/a3a4286d19524b538b70f1aa2a208ba6.png)

## 垃圾收集算法

> **已经能够确定一个对象为垃圾之后，接下来要考虑的就是回收，怎么回收呢？得要有对应的算法，下面介绍常见的垃圾回收算法。高效   健壮**

#### 标记-清除(Mark-Sweep)

* **标记**

**找出内存中需要回收的对象，并且把它们标记出来**

> **此时堆中所有的对象都会被扫描一遍，从而才能确定需要回收的对象，比较耗时**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\25.png?lastModify=1646720640)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/1521ea3d3ed64dfa814c22b06d43ccc6.png)

* **清除**

**清除掉被标记需要回收的对象，释放出对应的内存空间**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\26.png?lastModify=1646720640)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/5beab74efce64c5897d16f39db5e58f3.png)

`缺点`

```
标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程
序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。
(1)标记和清除两个过程都比较耗时，效率不高
(2)会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。
```

#### 标记-复制(Mark-Copying)

**将内存划分为两块相等的区域，每次只使用其中一块，如下图所示：**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\27.png?lastModify=1646720640)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/44cbc243638f4544917f0aa44e440c82.png)

**当其中一块内存使用完了，就将还存活的对象复制到另外一块上面，然后把已经使用过的内存空间一次清除掉。**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\28.png?lastModify=1646720640)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/b81f9e21abd84b0aaa102f27c4007986.png)

`缺点:`空间利用率降低。

#### 标记-整理(Mark-Compact)

> **复制收集算法在对象存活率较高时就要进行较多的复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都有100%存活的极端情况，所以老年代一般不能直接选用这种算法。**

**标记过程仍然与"标记-清除"算法一样，但是后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。**

> **其实上述过程相对"复制算法"来讲，少了一个"保留区"**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\25.png?lastModify=1646720640)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/331b1e9000d34b5093ed5b60a24e0402.png)

**让所有存活的对象都向一端移动，清理掉边界意外的内存。**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\29.png?lastModify=1646720640)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/874bcb40f36545279f8383408f50caf6.png)

### 分代收集算法

> **既然上面介绍了3中垃圾收集算法，那么在堆内存中到底用哪一个呢？**

**Young区：复制算法(对象在被分配之后，可能生命周期比较短，Young区复制效率比较高)**

**Old区：标记清除或标记整理(Old区对象存活时间比较长，复制来复制去没必要，不如做个标记再清理)**

## 垃圾收集器

> **如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\30.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/6f9efd6ca5b246629b25dbfec3d4a16c.png)

* Serial

**Serial收集器是最基本、发展历史最悠久的收集器，曾经（在JDK1.3.1之前）是虚拟机新生代收集的唯一选择。**

**它是一种单线程收集器，不仅仅意味着它只会使用一个CPU或者一条收集线程去完成垃圾收集工作，更重要的是其在进行垃圾收集的时候需要暂停其他线程。**

```
优点：简单高效，拥有很高的单线程收集效率
缺点：收集过程需要暂停所有线程
算法：复制算法
适用范围：新生代
应用：Client模式下的默认新生代收集器
```

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\31.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/e61e1fe960534b26b848708a753e05d6.png)

* Serial Old

Serial Old收集器是Serial收集器的老年代版本，也是一个单线程收集器，不同的是采用"**标记-整理算法**"，运行过程和Serial收集器一样。

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\32.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/fb0e283d58494cc9993355a1fa6ddbee.png)

* ParNew

**可以把这个收集器理解为Serial收集器的多线程版本。**

```
优点：在多CPU时，比Serial效率高。
缺点：收集过程暂停所有应用程序线程，单CPU时比Serial效率差。
算法：复制算法
适用范围：新生代
应用：运行在Server模式下的虚拟机中首选的新生代收集器
```

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\33.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/6ff517bede0043a2b37ff35c2b85f45e.png)

* Parallel Scavenge

**Parallel Scavenge收集器是一个新生代收集器，它也是使用复制算法的收集器，又是并行的多线程收集器，看上去和ParNew一样，但是Parallel Scanvenge更关注系统的吞吐量**。

> **吞吐量=运行用户代码的时间/(运行用户代码的时间+垃圾收集时间)**
>
> **比如虚拟机总共运行了100分钟，垃圾收集时间用了1分钟，吞吐量=(100-1)/100=99%。**
>
> **若吞吐量越大，意味着垃圾收集的时间越短，则用户代码可以充分利用CPU资源，尽快完成程序的运算任务。**

```
-XX:MaxGCPauseMillis控制最大的垃圾收集停顿时间，
-XX:GCRatio直接设置吞吐量的大小。
```

* Parallel Old

**Parallel Old收集器是Parallel Scavenge收集器的老年代版本，使用多线程和标记-整理算法**进行垃圾回收，也是更加关注系统的**吞吐量**。

* CMS

> `官网`： [https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector)
>
> **CMS(Concurrent Mark Sweep)收集器是一种以获取** `最短回收停顿时间`为目标的收集器。
>
> **采用的是"标记-清除算法",整个过程分为4步**

```
(1)初始标记 CMS initial mark     标记GC Roots直接关联对象，不用Tracing，速度很快
(2)并发标记 CMS concurrent mark  进行GC Roots Tracing
(3)重新标记 CMS remark           修改并发标记因用户程序变动的内容
(4)并发清除 CMS concurrent sweep 清除不可达对象回收空间，同时有新垃圾产生，留着下次清理称为浮动垃圾
```

> **由于整个过程中，并发标记和并发清除，收集器线程可以与用户线程一起工作，所以总体上来说，CMS收集器的内存回收过程是与用户线程一起并发地执行的。**

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\34.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/51a12718671b4d368a4721cb7cff4b5c.png)

```
优点：并发收集、低停顿
缺点：产生大量空间碎片、并发阶段会降低吞吐量
```

* G1(Garbage-First)

> `官网`： [https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html#garbage_first_garbage_collection](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html#garbage_first_garbage_collection)
>
> **使用G1收集器时，Java堆的内存布局与就与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，它们都是一部分Region（不需要连续）的集合。 **
>
> **每个Region大小都是一样的，可以是1M到32M之间的数值，但是必须保证是2的n次幂**
>
> **如果对象太大，一个Region放不下[超过Region大小的50%]，那么就会直接放到H中**
>
> **设置Region大小：-XX:G1HeapRegionSize=**<N>**M**
>
> **所谓Garbage-Frist，其实就是优先回收垃圾最多的Region区域**
>
> ```
> （1）分代收集（仍然保留了分代的概念）
> （2）空间整合（整体上属于“标记-整理”算法，不会导致空间碎片）
> （3）可预测的停顿（比CMS更先进的地方在于能让使用者明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒）
> ```

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\35.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/943614eae58f4649b9210c32fb2f8f98.png)

**工作过程可以分为如下几步**

```
初始标记（Initial Marking）      标记以下GC Roots能够关联的对象，并且修改TAMS的值，需要暂停用户线程
并发标记（Concurrent Marking）   从GC Roots进行可达性分析，找出存活的对象，与用户线程并发执行
最终标记（Final Marking）        修正在并发标记阶段因为用户程序的并发执行导致变动的数据，需暂停用户线程
筛选回收（Live Data Counting and Evacuation） 对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间制定回收计划
```

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\36.png?lastModify=1646736013)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/a04fb27a0ef64f8ca25a96eb95d44d71.png)

* ZGC

> `官网`： [https://docs.oracle.com/en/java/javase/11/gctuning/z-garbage-collector1.html#GUID-A5A42691-095E-47BA-B6DC-FB4E5FAA43D0](https://docs.oracle.com/en/java/javase/11/gctuning/z-garbage-collector1.html#GUID-A5A42691-095E-47BA-B6DC-FB4E5FAA43D0)
>
> **JDK11新引入的ZGC收集器，不管是物理上还是逻辑上，ZGC中已经不存在新老年代的概念了**
>
> **会分为一个个page，当进行GC操作时会对page进行压缩，因此没有碎片问题**
>
> **只能在64位的linux上使用，目前用得还比较少**

**（1）可以达到10ms以内的停顿时间要求**

**（2）支持TB级别的内存**

**（3）堆内存变大后停顿时间还是在10ms以内**

## 垃圾收集器分类

* **串行收集器**->Serial和Serial Old

**只能有一个垃圾回收线程执行，用户线程暂停。**

`适用于内存比较小的嵌入式设备`。

* **并行收集器**[吞吐量优先]->Parallel Scanvenge、Parallel Old、ParNeww

**多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。**

`适用于科学计算、后台处理等若交互场景`。

* **并发收集器**[停顿时间优先]->CMS、G1、ZGC

**用户线程和垃圾收集线程同时执行(但并不一定是并行的，可能是交替执行的)，垃圾收集线程在执行的时候不会停顿用户线程的运行。**

`适用于相对时间有要求的场景，比如Web`。

## 吞吐量和停顿时间

* **停顿时间->垃圾收集器 **`进行` 垃圾回收终端应用执行响应的时间
* **吞吐量->运行用户代码时间/(运行用户代码时间+垃圾收集时间)**

```
停顿时间越短就越适合需要和用户交互的程序，良好的响应速度能提升用户体验；
高吞吐量则可以高效地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。
```

`小结`:这两个指标也是评价垃圾回收器好处的标准。

## 生产环境中，如何选择合适的垃圾收集器

> [https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/collectors.html#sthref28](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/collectors.html#sthref28)

* **优先调整堆的大小让服务器自己来选择**
* **如果内存小于100M，使用串行收集器**
* **如果是单核，并且没有停顿时间要求，使用串行或JVM自己选**
* **如果允许停顿时间超过1秒，选择并行或JVM自己选**
* **如果响应时间最重要，并且不能超过1秒，使用并发收集器**

## 如何判断是否使用G1垃圾收集器

> [https://docs.oracle.com/javase/8/docs/technotes/guides/vm/G1.html#use_cases](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/G1.html#use_cases)

**JDK 7开始使用，JDK 8非常成熟，JDK 9默认的垃圾收集器，适用于新老生代。**

**是否使用G1收集器？**

```
（1）50%以上的堆被存活对象占用
（2）对象分配和晋升的速度变化非常大
（3）垃圾回收时间比较长
```

## JVM常用命令

* jps

> 查看java进程

```
The jps command lists the instrumented Java HotSpot VMs on the target system. The command is limited to reporting information on JVMs for which it has the access permissions.
```

* jinfo

> **（1）实时查看和调整JVM配置参数**

```
The jinfo command prints Java configuration information for a specified Java process or core file or a remote debug server. The configuration information includes Java system properties and Java Virtual Machine (JVM) command-line flags.
```

> **（2）查看用法**
>
> **jinfo -flag name PID     查看某个java进程的name属性的值**

```
jinfo -flag MaxHeapSize PID 
jinfo -flag UseG1GC PID
```

> **3）修改**
>
> **参数只有被标记为manageable的flags可以被实时修改**

```
jinfo -flag [+|-] PID
jinfo -flag <name>=<value> PID
```

> **（4）查看曾经赋过值的一些参数**

```
jinfo -flags PID
```

* jstat

> **（1）查看虚拟机性能统计信息**

```
The jstat command displays performance statistics for an instrumented Java HotSpot VM. The target JVM is identified by its virtual machine identifier, or vmid option.
```

> **（2）查看类装载信息**

```
jstat -class PID 1000 10   查看某个java进程的类装载信息，每1000毫秒输出一次，共输出10次
```

> **（3）查看垃圾收集信息**

```
jstat -gc PID 1000 10
```

* jstack

> **（1）查看线程堆栈信息**

```
The jstack command prints Java stack traces of Java threads for a specified Java process, core file, or remote debug server.
```

> **（2）用法**

```
jstack PID
```

## JVM死锁情况分析

```java
//运行主类
public class DeadLockDemo
{
    public static void main(String[] args)
    {
        DeadLock d1=new DeadLock(true);
        DeadLock d2=new DeadLock(false);
        Thread t1=new Thread(d1);
        Thread t2=new Thread(d2);
        t1.start();
        t2.start();
    }
}
//定义锁对象
class MyLock{
    public static Object obj1=new Object();
    public static Object obj2=new Object();
}
//死锁代码
class DeadLock implements Runnable{
    private boolean flag;
    DeadLock(boolean flag){
        this.flag=flag;
    }
    public void run() {
        if(flag) {
            while(true) {
                synchronized(MyLock.obj1) {
                    System.out.println(Thread.currentThread().getName()+"----if获得obj1锁");
                    synchronized(MyLock.obj2) {
                        System.out.println(Thread.currentThread().getName()+"----if获得obj2锁");
                    }
                }
            }
        }
        else {
            while(true){
                synchronized(MyLock.obj2) {
                    System.out.println(Thread.currentThread().getName()+"----否则获得obj2锁");
                    synchronized(MyLock.obj1) {
                        System.out.println(Thread.currentThread().getName()+"----否则获得obj1锁");

                    }
                }
            }
        }
    }
}
```

* jmap

> **（1）生成堆转储快照**

```
The jmap command prints shared object memory maps or heap memory details of a specified process, core file, or remote debug server.
```

> **（2）打印出堆内存相关信息**

```
jmap -heap PID
```

```
jinfo -flag UsePSAdaptiveSurvivorSizePolicy 35352
-XX:SurvivorRatio=8
```

## G1调优策略

> **（1）不要手动设置新生代和老年代的大小，只要设置整个堆的大小**
>
> **why**：[https://blogs.oracle.com/poonam/increased-heap-usage-with-g1-gc](https://blogs.oracle.com/poonam/increased-heap-usage-with-g1-gc)
>
> ```
> G1收集器在运行过程中，会自己调整新生代和老年代的大小
> 其实是通过adapt代的大小来调整对象晋升的速度和年龄，从而达到为收集器设置的暂停时间目标
> 如果手动设置了大小就意味着放弃了G1的自动调优
> ```

> **（2）不断调优暂停时间目标**
>
> ```
> 一般情况下这个值设置到100ms或者200ms都是可以的(不同情况下会不一样)，但如果设置成50ms就不太合理。暂停时间设置的太短，就会导致出现G1跟不上垃圾产生的速度。最终退化成Full GC。所以对这个参数的调优是一个持续的过程，逐步调整到最佳状态。暂停时间只是一个目标，并不能总是得到满足。
> ```

> **（3）使用-XX:ConcGCThreads=n来增加标记线程的数量**
>
> ```
> IHOP如果阀值设置过高，可能会遇到转移失败的风险，比如对象进行转移时空间不足。如果阀值设置过低，就会使标记周期运行过于频繁，并且有可能混合收集期回收不到空间。 
> IHOP值如果设置合理，但是在并发周期时间过长时，可以尝试增加并发线程数，调高ConcGCThreads。
> ```

> **（4）MixedGC调优 **
>
> ```
> -XX:InitiatingHeapOccupancyPercent
> -XX:G1MixedGCLiveThresholdPercent
> -XX:G1MixedGCCountTarger
> -XX:G1OldCSetRegionThresholdPercent
> ```

> **（5）适当增加堆内存大小**

> **（6）不正常的Full GC**

```
有时候会发现系统刚刚启动的时候，就会发生一次Full GC，但是老年代空间比较充足，一般是由Metaspace区域引起的。可以通过MetaspaceSize适当增加其大家，比如256M。
```

## JVM性能优化指南

![](file://E:\桌面\yzt\笔记课件\JVM\3天JVM训练营\资料+笔记\images\60.png?lastModify=1646833365)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1463/1646137467048/d86c29710a1a45ddbce7425a592a8528.png)
