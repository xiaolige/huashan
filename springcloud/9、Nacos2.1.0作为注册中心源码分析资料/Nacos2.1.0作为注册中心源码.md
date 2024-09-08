# 一、为什么有这课程

Spring Cloud Alibaba 新版本中Seata 1.5.2和Nacos 2.1.0 在性能和使用方面都有很大提升，这节课将从使用和源码的角度详细讲解这两个框架。

# 二、设计注册中心

## 1、分布式框架的注意点：三高架构

- 高可用

  高可用性（High Availability）通常来描述一个系统经过**专门**的设计，从而**减少停工时间**，而保持其服务的高度可用性(一直都能用)。

  解决方案：集群
- 高并发

  高并发（High Concurrency）是互联网分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计保证**系统能够同时并行处理很多请求**。 高并发相关常用的一些指标有**响应时间**（Response Time），**吞吐量**（Throughput），**每秒查询率**QPS（Query Per Second），**并发用户数**等。

  解决方案：

  - **垂直扩展**：**增强单机硬件性能**
  - **水平扩展**：
- 高性能

  高性能（High Performance）就是指**程序处理速度快，所占内存少，cpu占用率低**。

  解决方案：

  指程序处理速度快： 这里设计我们数据存储结构、访问机制、集群同步

## 2、注册中心的设计

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/370b76a9fa1a4d39ba69f2f9de637121.png)

- 服务注册
- 注册表结构设计
- 服务发现
- 服务订阅
- 服务推送
- 健康检查
- 集群同步：设计到数据同步，数据同步我们有哪些协议 raft 、distro、ZAB

# 三、Nacos作为注册中心源码分析

## 1、项目准备

* 客户端项目：msbshop-parent
  注意版本的对应

  ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/bc779156c7674e4c8dd5a973623aba7d.png)
* 服务端项目：nacos-2.1.0

  下载对应的nacos，进行编译

  ```
  mvn clean install -DskipTests -Drat.skip=true -f pom.xml
  ```
* 启动源码服务时候指定参数

  ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/b2c6ac0894cb46668f68675612a8531a.png)

## 2、服务启动入口

#### 2.1 整体流程图

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/42196557ce97408d8902b1d031dde65d.png)

#### 2.2 源码分析

通过学习Nacos1.4 我们知道了我们的注册对应的类是NacosNamingService，那我们Nacos2.1.0是不是也是一样的呢？

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/97a2094a1f5c40bfbfc507181d87a45c.png)

通过上图我们发现里面也有注册的信息，那我们打上断点看一下是否到这里，我们在对应注册方法打上断点，然后debug启动，如图 ：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/78625fabb4034444a857ec23608de01e.png)

我们发现服务启动后他确实通过这里，所以这里是注入的入口，那他是从哪里来这里的呢？

我们想nacos是和Springboot整合，那可能使用了SpringBoot的自动装配

我们查看我们引入的Jar包spring-cloud-starter-alibaba-nacos-discovery.jar 查看里面spring.factoris文件

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/0cd34efd3c21495fbfdf36684f8fa268.png)

我们可以查看自动装配类，在这里我们可以通过名称来推断那个关于注册自动配置类，NacosServiceRegistryAutoConfiguration应该和我们注册相关，我们进入看一下，导入以下类

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/dbe557c92ad044dcb37a450c4decaf7d.png)

我们可以通过上面debug端点看一下堆栈信息，看涉及到那个类

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/bde7371d0fbd4e9d89ba2dfdd2dd3705.png)

他的默认实现类是NacosAutoServiceRegistration

NacosAutoServiceRegistration 实现了ApplicationListener接口，监听事件WebServerInitializedEvent（spring核心方法refresh的完成后广播事件）

#### 2.3 总结

找入口的方式：自动装配类 spring.factories

事件驱动：NacosAutoServiceRegistration实现了applicationListener接口

## 3、实例注册

#### 3.1 整体流程图

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/ab4e3f1de593405ca7d5a623d411566a.png)

#### 3.2 客户端

##### 3.2.1 源码分析

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a6f5987d51bf4f5b8e89d3d09c534eb1.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/35cdd40aaeec42bfb67f2c24b654b837.png)

我们的初始化代码如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/d3a9f62563e3418e85165639c28d366e.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/82ce6c987a9d44a280f218078a2070d5.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a791f314888641dd94685a2c82f840c4.png)

上面是通过实例的参数ephemeral值来判断是否是grpcClientProxy还是httpClientProxy，我们在他的实例化的位置能判断ephemeral的值

我们根据堆栈信息去看找到instance实例化的位置

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/ee46e3af25c4458a9a5817541f753dbd.png)

具体实例创建

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/190b5d83863b4c14ad6282ece8a790ef.png)

分析NacosDiscoveryProperties![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/8d5c5f62e04f4f6fa7bd0d3ffb95c259.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/31e918a4877b4a7294d30ec2c31888d9.png)

由此我们知道这里使用grpc客户端端和服务端通信，同时我们能得出结论：我们

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/beb4a181466f47a8b2129b8d47cda547.png)

grpcClientProxy的调用

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/79f06e07b41b45df94a70d1ab0bf84df.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/07a40ee2edaa4476a50a976d38856c46.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f25d21d3e6a6463591db41987e56d81f.png)

##### 3.2.2 源码总结

* 判断变量  1、 debug  2、 全文搜索 定位赋值位置
* 通过ephemeral的值判断是grpc通信，还是http通信，通过这我们能判断ap模式是用的grpc模式，cp模式是用http通信
* 判断服务端处理类的方式，我们可以根据请求参数，找对应服务端的处理类（由于开源框架都是规范的，一般都是根据请求参数来命名，所以可以采用这种方式）

#### 3.3 服务端

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/d0092239af1441f89574b368525acdb5.jpg)

##### 3.3.1源码分析

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/26a80427e44a401d80c55023bfe3c67c.png)

这里我们注意我们实际注册的应该是对应的实例，而不是服务，服务包括多个实例，具体上的实例才有对应的ip和port

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/e98a5da3c2c64154a139342bd4b90149.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/c0ebbff5ab9e4ac3a1e6678b1779be02.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/ed3840857d874a63bc4eb9464d68dfde.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/8aab301210e94cb4870039b154671817.png)

问题：这里一个服务对应一个实例，我们知道一个服务应该对应多个实例，为什么这里对应一个实例呢？ 他是怎样处理的？后面我们进行解答

对事件ClientRegisterServiceEvent的监听，我们可以通过全文搜索来看，哪里对应的处理的

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f0fb4c744366447abfcc2f19d1109c26.png)下面进行事件监听进行处理：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/b7dc8941859c4c4580708a2b9fdb5861.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/b8aa6d23a3a34b93940b4f63d56a480d.png)

publisherIndexs就是注册表

##### 3.3.2 源码总结

1. 分清实例和服务的关系： 我们实际注册的是实例 ，一个服务包含多个实例
2. 这里注册实例会注册到我们Client中，Client有个Map,key：是service value是对应的一个实例，也就是一个Client对应一个服务的具体实例
3. 我们发送事件后来处理注册表，注册表结构是ConcurrentMap&#x3c;Service, Set&#x3c;String>> publisherIndexes
   里面key:是服务，value对应Client的clientId

## 4、注册表

### 4.1 注册表结构分析

下面是我们nacos2.1.0的注册表

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/b5d9aa23dd514bcbae9f2c8f083a8187.png)

这里是Nacos1.4x的注册表

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/ee2c55fd719f4d74be206947b834b7f0.png)

### 4.2 总结

通过Nacos1.4 和Nacos2.1 版本的注册表结构会发现，1.4 比较复杂，而2.1是比较简单，这样2.1的加锁的地方要少一些，所以2.1的注册表的结构性能要好，所以说我们可以总结一下,Nacoa2.1比Nacos1.4性能好的原因：

1. Nacos2.1 使用的是GRPC性能要好一些
2. Nacos2.1 表结构简单，所以加锁的地方要少一些，性能更好一些

## 5、服务发现

### 5.1 客户端

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a2292a67de4245c98de2e9de9441bd0e.png)

#### 5.1.1 源码分析

我们需要从客户端找到服务发现的入口，我们注册的入口类是NacosNamingService，那我们服务发现入口应该也在这里：我们看一下有两个方法：getAllInstances和selectInstances

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/92f8bc84bc71459a8632fa81700cde50.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/82c9cf932a2c4de1901ce9d509267890.png)

那具体是那个方法，我们可以在每个方法里面打上断点，然后debug启动后，进行访问

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/661d1b8b2ef14dd38f0158ba3b96f5a3.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/106051abdf674438add0b5c30aab616a.png)

发送请求

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/e524241b700045d48d9aee66096490ed.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/5456027ed9314a87adbcfc64bb993fca.png)

从上图可以判断他是通过selectInstances来获取数据的

我们通过堆栈信息我们能发现，他的调用路径是通过ribbon最终调用selectInstances方法。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/c6eb6a13445b47f588759524a24cf5a0.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/3aae38ce59f146b7b65a7e285fc987d3.png)

key1: 从缓存中获取数据

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/bf7db4b84d0a4751a8a54420fa0a15de.png)

这里注意，他是从缓存中获取数据，那他一定有定时任务处理这里的数据，否则他就会有脏数据的问题，带着这个问题我们继续学习。

key2：进行订阅

我们服务启动第一次一定是空的，所以我们进行订阅，当定于的clientProxy，具体是那个值呢？

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/db8290726a934d808fdee0ef7b149d93.png)

我们能确定clientProxy的具体实现类是NamingClientProxyDelegate，好我们看一下具体

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a5c1ed84878c4072a814f73376e3a5ac.png)

key1:定时任务

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/be300ef20d7847e8ad6e6edf24d50948.png)

这里的UpdateTask一定是个Runable，所以我们看他的run方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/cdd3431d10f1466d8ce4c5154cc2e124.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/bd0414d7d463487e9ed5cd063d2e088a.png)

总结上图内容：从缓存获取，如果没有则发送请求获取，如果有则判断数据是否超时，在finally里面对应的内容是这个任务每6秒执行一次，如果失败就会扩大两倍时长，最大是一分钟

里面更新缓存中内容如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/751501f16d574f369c5764ddb207c126.png)

这里先写内存后写磁盘，那磁盘数据什么时候获取呢？

我们在ServiceInfoHolder构造方法中发现？

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/1327f355086741d6be3662bc87af818a.png)

那我们看一下服务发现对应代码

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f8c042c29f6d42c1988f5633d21f251e.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/1c322b1b1e5743d986418ab9c57b6d7d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/bc7b2cbb7728445ebb707db39a6b27b1.png)

#### 5.1.2 源码总结

- 服务启动时候读取磁盘中数据放到缓存
- 读取磁盘数据 如果没有则发送请求获取数据，然后写到缓存
- 读取磁盘数据，如果有则判断时间是否过期，过期则发送请求，写到缓存
- 读取数据是一个定时任务每6秒执行一次，如果失败就会延长，但最大时长是1分钟

### 5.2 服务端

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/508e673fc5c842ed9e78bdbd0a276714.jpg)

#### 5.2.1 源码分析

从上文中我们可以分析出我们服务端处理类应该是ServiceQueryRequestHandler

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/6b618486762449cab366071dde77bc1b.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/d092f7a56b5e411a971b1a588c934e34.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/979eb1b16dc24940bbf46031905936af.png)

从上图可知getAllInstancesFromIndex是重点，它是获取我们对应的实例，来我们重点分析一下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/c957c4ab1f934d4e96b73d4ce8f06534.png)

1、获取对应的clientId，这一定是在我们的注册表中获取的，注册表示个Map ,key:是服务名称，value:是对应client的set ，不理解的可以参考一下我们的注册表

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/4482a3bc7faf438187cd509208e41529.png)

2、获取对应实例这里我们详细看一下

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/03973e7b54f54753a17438e8da952f84.png)

这里可以和我们注册表中内容详细看一下

#### 5.2.2 源码总结

这里主要是从注册表中获取数据，所以理解这里需要看一下我们的注册表中，各个数据之间的关系

## 6、服务订阅

### 6.1 客户端源码

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a2292a67de4245c98de2e9de9441bd0e.png)

源码可以参考我们服务发现对应的源码

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/644ccf6f847f487ba6485e24901825ae.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/c47d1abdf562412b98089fb5c0820163.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/bd059a88087e4da8a62c9d69115bae0c.png)

### 6.2 服务端源码

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/8322e18df1c74754baf9e4a7cf0d00ef.jpg)

#### 6.2.1源码分析

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/1419b31f593a4ccdaf842e239c1baf62.png)

key3: 查询服务列表的信息中，会调用serviceStorage.getData(service) 来获取服务的实例，这个和我们服务发现的服务端源码是一样的，这里就不在重复

key4:进行订阅

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/8e5330fb69d3492184f9d983aeabf760.png)

key2:把订阅者和服务进行关系绑定

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/48579359ac5446f9bf179cb5dc914aca.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/304d290e52e34c5ab4e623ec8e38848c.png)

这里和注册相同，每个客户端对应一个Map,Map里面key:是服务  value:就是这个服务对应的实例

key3:发送订阅事件， 我们全文搜索ClientSubscribeServiceEvent 查看处理事件位置

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f063c749430148248dfe747fd53dea55.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/ff9cc37010024cc48992476dec170078.png)

更新订阅表

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/b9098a26573f4ee5bc4a2e2dd2816863.png)

#### 6.2.2 源码总结

我们的订阅很简单首先是获取对应订阅这个服务的实例，用于返回，然后进行订阅，订阅的信息是我们对应发起定于服务者和被订阅服务会以map形式放到我们的client，然后发送一个事件，这个时间就是更新我们的订阅注册表

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/6af0125c384845f4a0967cbe5988cb1b.png)

## 7、服务变更推送

这里注意我们推送有两种一种是服务变更推送，一种是订阅推送

服务变更推送：对于一个服务来说，订阅者有好多，我们在订阅表中能看到ConcurrentMap&#x3c;Service, Set&#x3c;String>> subscriberIndexes，能获所有订阅者，然后进行推送

订阅者推送：此时服务已经稳定，我这里增加一个msb-order，那对于msb-stock又增加一个订阅者，此时我们应该将msb-stock对应的实例推送给具体新增的实例

### 7.1 订阅推送

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/c7c88a6fd9ec45baad4a0ee73f3572cf.jpg)

#### 7.2.1服务端源码分析

客户端在订阅的时候发送事件更改注册表后，会再发送一个事件，这个事件就会获取对应的服务实例，然后通知订阅者获取对应的服务的实例，这里和上面我们直接订阅返回对应数据有点重复，这里我们可以注意一下，接下来我们看一下源码。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/5020e9b325264373a3533ed5fd953609.png)

全文搜索ServiceSubscribedEvent查看处理事件的地方

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/ca0df48ab73f45d994dcb2721cd136b0.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/d5496d66597a47498e12daf83e4b4462.png)

我们进入addTask方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/2ed6461dd30d4a8faf868f1cc33cccc5.png)

我们想这里先从map中获取如果有则合并，没有则放进去，那我们想一定有个地方从这里来获取这个任务来处理；

我们看一下这个类的构造方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/78edd31ad42546c7aaa549d8b5b12ceb.png)

这里有个定时任务来处理我们看一下

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/927f00185e9a431da2584642231a710d.png)

我们进入处理任务

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/0a0c6e2b0d6744d4b30f61580f852b33.png)

这里的processor.process的处理方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/422725019753408c94e4c940189ce5e4.png)

我们分析一下上面到底是那个方法，1、我们可以通过debug 2、我们可以猜测，我们想我们处理的任务是PushDelayTask ，所以我们用它PushDelayTask来搜索一下

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a92b8842e1c04e008619193763ec62c0.png)

所以处理类应该是PushDelayTaskProcessor

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/5d0f06944b7e4ed7a8c1997e972db0cd.png)

PushExecuteTask是实现Runable的线程，那重点应该是他的run方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/6daa30a8a3a94c78af3b4cf5a476f706.png)

进入对应的run方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/eb2663141a394a9a936db02242cd9381.png)

key1:获取订阅这个服务的客户端ID

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/5489002a56a042699a1c076ac842d2f6.png)

上面是判断是否是全部推送，如果我们有指定的ClientId就不全部推送，如果没有我们就全部推送，我们这是订阅推送，所以有要推送的客户端

key2:进行任务处理

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/77e823e83b31499bbaa9f580a2e2316e.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/7254cff9873a4771ace060bb5724a210.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a72afdb86a434d0d91d0757aec2d372a.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/7027251733aa48f7b5888cb05f5f3ad7.png)

#### 7.2.2 客户端源码分析

服务端发送请求参数是ServerRequest，那我们怎样在客户端找到对应的处理方法呢？ 还是老办法，用ServerRequest在客户端进行搜索

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/fff1604f2b964bb2a6c8036169a1a5cf.png)

这个ServerRequestHandler是个接口，我们找他的实现类，如下

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/10e54c661a624f5f861578026944907b.png)

那具体的实现类，我们推理可以知道他一定是NamingPushRequestHandler

好我们来分析一下NamingPushRequestHandler

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/cf7e8ad210f6495eae2541d66443fe55.png)

#### 7.2.3 总结

我们新增服务订阅在更新完注册表会发布一个事件ServiceSubscribedEvent，NamingSubscriberServiceV2Impl.会监听事件，然后把订阅到的数据进行推送过去，这期间真正的工作任务是PushExecuteTask他是一个线程，同时注意这里是订阅，所以我们推送的时候，直接推送给新增订阅者就可以

### 7.2 服务变更推送

服务变更推送和订阅推送是一样的只是推送的对象不同，我们订阅推送是给新增订阅者进行推送，服务变更推送是给所有订阅这个服务的推送

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/9908719e64dc4765ade45de9200db571.jpg)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/9322b64b71044538b022f51200038f75.png)

## 8、心跳机制

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/42e51ba25c394259aff315ccffe78c33.jpg)

### 8.1 源码分析

我们进入ConnectionManager里面的start方法中，这个方法上有@PostConstruct，也就是构造方法执行完毕就执行这个方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/bced28ccc13b4f8c99a409cdad964f2d.png)

上图中我们可以知道它里面线程是每3秒执行一次：

由于里面方法比较多，这里我们可以看一些关键的点：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/619e5ca6937f404bbe1ae85428785500.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/c65bf5fad68a49599d5fd6b188ea37cc.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/11b28027d4c547a18ee6fcaaf962f2a4.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/115580554f034e95a2881f1097d6eaa6.png)

key5:注销

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/07d6b7de0a744414ba9a893fb214d201.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/efafefd64c5c497285272ec968887ca0.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f8fb000d21e640ea97bef9b9a9001059.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/e16d0576c28a464592842bf1322c5d71.png)

发布事件ClientDisconnectEvent

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f6dd3f0242ef42ddab76e3258a3d0a6b.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/e1d1d0e4f4fd4a70848b208a7e28fe0f.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/8e76ae3ec3384ceb8c43760e8f70b97c.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/93b47f448108459b9d232ce116fea190.png)

### 8.2 总结

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/6a08c0144e2b44e0a395a78b9dd2d0b3.png)、

## 9、数据同步

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/b2e076ff364c448d8c3158b2d4769ed7.jpg)

### 9.1 源码分析

我们在实例注册的时候就应该进行集群同步

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/ec9325643c7546449198829484fb5297.png)

全文搜索ClientChangedEvent查看哪里处理

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/cea69f1c922d406cb722e18f73d9bbb6.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a58d4e4856e24fb39a22f448b6a4d055.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f9b18a0216f040fcba74a975d9d9dad8.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/13f2426ca52d4e128554d18123e9faee.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/6f33f8beec0947b289adc464f6314dd5.png)

我们进入addTask方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/9fd694498ea748ef834c1ea17986613e.png)

进入NacosDelayTaskExecuteEngine的构造方法里面，启动一个定时任务ProcessRunnable

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/8acbd4198f64437d97195e1892672549.png)

查看ProcessRunnable的run方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/2dcb876d7f8f426e8059e929738eb6c8.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/b45440a90db340daa59cd9839e81cd8a.png)

我们处理的任务是DistroDelayTask ，所以说我们查看具体方法如下

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f43db064b4f24475aeafa837555d03d5.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/de4e110be3744e16b1de2a45d1d5ee93.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/7a9741167ddb4d989d0c294c0d35a4de.png)

DistroSyncChangeTask是具体的任务我看一下他的run方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/59de275cbc34425d96906faee2b9fbc8.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/329c48086b4d4897a2d415a58f3f3cef.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/e9717df0cf0b4094898af65b2ee4b4b6.png)

处理集群同步

从上图中我们知道集群同步的请求是DistroDataRequest， 那我可以在服务端全文搜索

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/04c8549da90b4fb888de356eb33487d9.png)

处理数据同步

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/1a13e83f8b3c429380e807d7b8b28ee9.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/14d1971e1d83457ea92c40f9809e60eb.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/0306ec408b8c4777a19684e855fdf079.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/4076f20e99ac44638378931d30c599b0.png)

新增实例

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/7f353e5b2a624b64b1453ef85ebc5cec.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/e22d42e3ba8e4ddbacfd0e80fb75240d.png)

### 9.2 总结

这里集群同步时候，我们采用的异步处理，这里和我们服务变更推送类似

## 10、GRPC源码分析

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/94e6b68fb2ac43f58c66d49868371094.png)

### 10.1、客户端源码分析

我们找到客户端注册的代码：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/6b30796225b1466793aca964257de9a2.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/d0bca279153742e492343a42b91ff152.png)

通过他初始化实例，我们知道他他真正的调用方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/8c696604aa0d47f6bdde3a3525b8eea0.png)

通过方法getExecuteClientProxy需要确定具体代理类 grpcClientProxy

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/149895442eb648d592db37b2424dd52d.png)

我们查找grpcClientProxy的初始化的地方。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/9403d79e190846659e258c88a895a869.png)

那NamingGrpcClientProxy应该grpc的核心逻辑

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/aca98f4b5b8e4996ac8fe991a0a0c8f6.png)

我们进入start方法：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/0ee522ee6a3443ff9b0391cd07dc8bfa.png)

上面方法分为两部分：

key1:是将处理的Handler放到对应的List<ServerRequestHandler> serverRequestHandlers  那后面一定是在客户端处理请求的时候，从哪这里面拿到对应的Handler进行处理

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/84eaa68eed2742e8bb1e2cc975003537.png)

key2: 启动这里 这里是关键，我们进来看一下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/334ebda68a814706a3120ca78d762c19.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/5d1b8d2d678341048eaa9b1e5e7b74a0.png)

上图是简单描述：我们可以看一下第4、5、6、7步

key4:选择一个服务

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/e1be63a9a9194cfe834ac8a7556eef2b.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/7eefc90a0c7f42e7bf3dbe03e7c4e377.png)

key5:连接服务

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/6baf39ab23a348d596a74d1d2a2ea744.png)

key6: 发送事件到队列， 我们可以全文搜索哪里处理这个队列

线程池里面的一个线程正在处理，如

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/35424d4df43c435fbdaef72756128b01.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/71135fd687eb4d649f0ae35530a5c802.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/340903e3b32741329263e903e56d284f.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/b42b7b0e795e449b967d233cde27d183.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/4236b2c431a94bb69fcca0805a611a2e.png)

key7:

如果前面同步连接都失败的话，我们就进行异步连接

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/452d5387f0144fba93e1800922e86891.png)

我们全文搜索就会发现在上文中的第二个线程中会进行处理

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/5aa6fa69e04340f5abd35e62419301f2.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/8b7381f0084e4ed7a134c6c0e66ca5e8.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f7f9543c9ca0427f857eb92815228c0c.png)

总结：在连接成功还是连接失败后都是异步进行处理，我们可以参考这种方式

### 10.2、服务端源码分析

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/e3af7e0873544b8a8f6a7865109fc943.png)

服务处理rpc的请求，那我们可以进行猜测一下服务端进行搜索RpcServer，如下

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a44a1a47239441858c3c266ee9b2a8b5.png)

那BaseRpcServer应该是处理RPC的请求，我们看一下对应代码

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/13fe900e8f6647c681f7ebfa98845a2d.png)

通过名称我们应该知道他是启动server，那我们查看哪里调用他

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/ecd2d0abb2f84265b85602bb4ed5d4b5.png)

通过上图我们知道，他是构造方法之后我们进行服务启动

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/aaa3731c12f841508fc24a2a5ce5a216.png)

key1: 增加拦截器 ，这里主要获取一些远端Ip+port信息

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/4f525cb4d9f64bb6b940a58c4fb50838.png)

key2:  这里面重点是建立连接和处理请求

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/ee5b2a41eabb4f37a5f51d603c57aa2a.png)

key2.1 处理请求

我们想我们Springmvc处理请求的时候，我们是根据路径判断对应的controller，这里我们的请求应该是那个具体的handler呢？ 我们是根据type，这个type其实就是请求类型

如下图：获取对应type ,然后根据type获取对应的的RequestHandler

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/4a78359481fb451091d0af8d03df1d83.png)

我们进入getByRequestType

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/9f80113e5ba04c6e93c0fefae031fd2d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/f3cb3967454d4b06b002c6daa695968f.png)从上图我们知道这里是将相应handler以map的形式进行存放，那这个key我们通过debug知道他对应的值，为请求参数的的名称

那什么时候进行初始化的：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/0b6dc0cba5b84fa8978102a37a3f1a9f.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/0f8fe3683e674e2ca594bf526d09b5f5.png)

那Handler对应泛型的第一参数类型名称是什么，那我们拿InstanceRequestHandler来举例：也就是InstanceRequest

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1670652088093/a96da3e0de7e4f2797a1bddb2c4dc1a8.png)

总结：我们处理实例就是用的监听器来获取对应的所有实例，然后以map处理，所有请求从这map中拿取对应的对象。


