# 一、SpringCloud简介

## 1、SpringCloud是什么

- Spring Cloud是一系列框架的有序集合，这些框架为我们提供了分布式系统构建工具。

## 2、SpringCloud包含那些项目

| 项目           | 项目名称                                         |
| -------------- | ------------------------------------------------ |
| 服务注册于发现 | Alibaba Nacos、Netflix Eureka、Apache Zookper    |
| 分布式配置中心 | Alibaba Nacos、Spring Cloud Config               |
| 网关           | Spring Cloud Gateway、Netflix Zull               |
| 限流熔断器     | Alibaba Sentinel、Netflix Hystrix、 Resilience4j |
| 服务调用       | RestTemplate、Open Feign、Dubbo Spring Cloud     |
| 负载均衡       | Spring Cloud LoadBalancer、Netflix Ribbon        |
| 消息总线       | Spring Cloud Bus                                 |
| ...            | ....                                             |

## 3、SpringCloud版本选择

https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E

| Spring Cloud Alibaba Version | Spring Cloud Version       | Spring Boot Version |
| ---------------------------- | -------------------------- | ------------------- |
| 2.1.4.RELEASE                | Spring Cloud Greenwich.SR6 | 2.1.13.RELEASE      |

| Spring Cloud Alibaba Version | Sentinel Version | Nacos Version | Seata Version |
| ---------------------------- | ---------------- | ------------- | ------------: |
| 2.1.4.RELEASE                | 1.8.0            | 1.4.1         |         1.3.0 |

# 二、Nacos安装以及编译

## 1、下载源码

解压进入目录中进行maven编译

```pom
mvn clean install -DskipTests -Drat.skip=true -f pom.xml
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/a7aad72f3e894e8fbddee6cd9daa379d.png)

> 注意：编译的时候可能需要你自己指定jdk版本，可以修改maven配置文件conf/settings.xml
>
> ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/f1fdb66f9a5e40e280ba734e496ff814.png)
>
> ```xml
> <profile>  
> <id>jdk-1.8</id>  
> <activation>  
>    <activeByDefault>true</activeByDefault>  
>    <jdk>1.8</jdk>  
> </activation>  
> <properties>  
>    <maven.compiler.source>1.8</maven.compiler.source>  
>    <maven.compiler.target>1.8</maven.compiler.target>  
>    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>  
> </properties>  
> </profile> 
> ```

## 2、源码单机启动

- 将jdk版本都设置为jdk8
- 设置参数

```xml
 -Dnacos.standalone=true
```

## 3、单机启动服务

- 下载nacos服务

  https://github.com/alibaba/nacos/releases

  ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/b5029ca5d167466bb61be38f98d0d6e6.png)
- 解压进入bin目录
- 执行命令

  ```shell
  startup.cmd -m standalone
  ```

## 5、修改startup.cmd

将MODE模式改为standalone，这样下次直接双击startup.cmd就可以了

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/50e8bb614a2944459922d92342423788.png)

# 三、Nacos服务领域模型

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/f5f315ce86494c35bc38bab9d0b52da1.png)

service->cluster-> instanc【之所以他会这样设置就是为了大的互联网公司，多集群垮机房提供了解决方案。但我们小公司一般都不需要这样。】

Namespace:实现环境隔离，默认值public

Group:不同的service可以组成一个Group，默认值Default-Group

Service:服务名称
Cluster:对指定的微服务虚拟划分，默认值Default

Instance:某个服务的具体实例

Nacos服务注册中心于发现的领域模型的最佳实践。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/0bbad64c99b04261bf9dc1933d863a10.jpg)

NameSpace: 是我们生产，开发和测试环境的隔离。测试访问测试，生产访问生产。

Group:比如说有一类服务。他们都是为了交易而服务的。比方说我们的订单，比方说我们的支付。这个在服务注册中心的场景中并不常用。

Service:下一个层级就是service，同一个group有多个service，再服务下面就是集群的概念：

Cluster:比方我们可以有北京的集群也可以有上海的集群。那这个集群的概念意义是什么？ 我们可以设想这样的一个场景，比方阿里只在杭州部署一个淘宝的集群，那我们北方的人民去访问，是不是就会相对较慢，一般都会再南方和北方都设置两个集群。当北方人民访问的时候一般我们都访问北京这个集群的服务，其他的服务都会在北京的集群里面互相调用，而不会垮cluster进行访问，这样在同一个数据中心访问都是比较快的。这就是他的意义。

Instance: 这个就是实例，并且是多实例的，这样防止单点问题，从而实现高可用，当北京集群坏了，他会可以访问上海的集群。当一个实例挂掉，另一个实例会替代，当一个集群挂掉，另一个集群会替代上。这样就保证了高可用。很多公司宣传5个9或者4个9的可用，那么全年停机的时长不会超过固定的时长。

# 四、Nacos的使用

## 1、 正常使用http客户端调用

我们下订单的时候的调用：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/cb1801af00ec4560ba525dd6f6d5a242.png)

1、讲解单独调用的图
2、然后运行一下这个demo
3、提出问题
4、进行改造

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/291eb48cdd134d17894d9d3c4b87334d.jpg)

## 2、引入nacos

改造msb-stock

1、父pom引入依赖

```pom
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
```

2、启动注解

```java
@EnableDiscoveryClient
@SpringBootApplication
public class StockApplication {
    public static void main(String[] args) {
        SpringApplication.run(StockApplication.class);
    }
}
```

3、增加配置

```yaml
spring:
  cloud:
    nacos:
      discovery:
        service: msb-stock
        server-addr: localhost:8848
```

改造msb-order

2、启动注解：

```java
@EnableDiscoveryClient
@SpringBootApplication
public class OrderApplication {

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class);
    }
}
```

3、增加配置

```java
spring:
  cloud:
    nacos:
      discovery:
        service: msb-order
        server-addr: localhost:8848
```

4、改造代码：

```java
  @GetMapping("/order/create")
    public String createOrder(Integer productId,Integer userId){
        String result = restTemplate.getForObject("http://msb-stock/stock/reduce//" + productId, String.class);
        return "下单成功";
    }
```

5、现在我们访问msb-stock ，但是restTemplate并不知道怎样调用

```java
@GetMapping("/order/create")
public String createOrder(Integer productId,Integer userId){
    // 此时restTemplate并不能识别msb-stock所以他不能进行调用
    // RestTempLate调用需要一个负载均衡器 1、 获取msb-stock对应服务列表 2、选择一个去调用
    // RestTemplate扩展点clientHttpRequestInterceptor
    // 我们有个组件ribbon 实现了这个扩展点 LoadBalancerInterceptor
	// 他做的事情就是将msb-stock:替换为：localhost:11001
    String result = restTemplate.getForObject("http://msb-stock/stock/reduce//" + productId, String.class);
    return "下单成功";
}
```

分析一下LoadBalancerInterceptor的拦截器Interceptor

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/9be010bb61624c1e9bddee2f5c108fc9.png)

我们可以进行如下改造：

```java
@Autowired
LoadBalancerClient loadBalancerClient;

@Bean
public RestTemplate restTemplate(){
    RestTemplate restTemplate = new RestTemplate();
    restTemplate.setInterceptors(Collections.singletonList(new LoadBalancerInterceptor(loadBalancerClient)));
    return restTemplate;
}
```

当然我们也可以进行如下改造：直接加入注解@LoadBalance后面我们会讲

```java
@LoadBalanced
@Bean
public RestTemplate restTemplate(){
    return new RestTemplate();
}
```

# 五、Nacos注册中心的原理

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/706cf73694cb46cda91be6681142c94b.png)

首先服务在启动的时候会将数据注册到注册中心，这样就服务只要启动就可以对外提供服务了， 那么同时他也要维持一个心跳，心跳的意义就是某个服务挂掉，我肯定让注册中心知道，如果服务挂了还没有告诉注册中心，那么注册中心会引导客户端调用挂掉的服务体验就不好了，所以注册中心重要的概念就是和我们的服务维持一个心跳，来实时的监听我们服务的状态，当服务挂掉就会将其从服务列表中踢掉，接下来肯定还有服务的调用者，他在调用服务之前一定会到注册中心拉取服务列表，获取可用的服务，并且还有一个定时任务来定时拉取服务列表，以保持服务列表的最新状态，其次还有更新的功能，这个功能就是当服务挂掉的时候通过心跳他会发现这服务不健康了，他会更新本地的服务注册表，同时将所有订阅这个注册表的服务进行更新。



# 六、集群搭建

## 1、解压Nacos

解压Nacos，复制三份

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/164477700a5f4eedb5ae6d26c809f72a.png)

## 2、配置数据库

具体位置在nacos目录下的conf中，这里的操作和之前是一样的，我们可以直接打开这个文件然后拷贝到数据库中执行，当然也是要创建数据库使用数据库然后在复制脚本内容，执行即可。

```mysql
create database nacos_config;
use nacos_config;
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/75438064394a4a81a5b336b80658e149.png)

## 3、修改application.properties配置文件

修改每个服务数据库链接

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&serverTimezone=UTC
db.user=root
db.password=123456
```

修改每个服务端口

```
server.port=8848
```

## 4、修改集群配置文件

将cluster.conf.example 改为 cluster.conf，并添加对应服务集群Ip:port,

其他文件夹也要修改

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/76d3090e3651495cb40e18bfb62377fa.png)

## 5、集群启动

```shell
D:\toolsDev\springalibaba\nacos cluster\nacos-8848\bin\startup.cmd -m cluster
D:\toolsDev\springalibaba\nacos cluster\nacos-8868\bin\startup.cmd -m cluster
D:\toolsDev\springalibaba\nacos cluster\nacos-8888\bin\startup.cmd -m cluster
```

## 6、浏览器访问一下Nacos

## 7、安装Nginx

修改nginx.conf

```java
worker_processes  1;

events {
    worker_connections  1024;
}

stream {
      upstream nacos {
        server 192.168.1.11:8848;
        server 192.168.1.11:8868;
        server 192.168.1.11:8888;
      }


     server {
        listen  81;
        proxy_pass nacos;
     }
}
```

启动nginx

```shell
start nginx.exe
```

重新加载

```shell
nginx.exe -s reload
```

## 8、进行访问

http://localhost:8848/nacos

## 9、项目链接

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/15647/1660380661024/625ca1aabd514674a0addb730f84bf62.png)
