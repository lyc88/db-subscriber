# db-subscriber
## 介绍
数据库订阅中间件，当订阅的数据库/表发生数据改变时，主动推送消息给业务端或者直接实现业务需求。 主要使用场景：
  1. 对Redis等缓存的更新。
  2. 多数据库数据同步。
  3. es数据同步。
  4. 其他需要监听数据库变化的业务。
## 优势: 业务端一个注解搞定，无业务侵入。
### 示例场景：运营人员在控制台手动开启一个活动，服务端需要及时将开启的活动设置到缓存中。
  传统做法：控制台更改时，发送MQ消息通知给服务端，这样控制台的业务就有了侵入性，而且有可能不会做到及时更新。</br>
  db-subscriber能够将这类的数据同步，缓存更新的业务进行解耦，添加一个@Subscriber注解，即能感知到某条数据的变化。
## 开始使用
### 环境
  1. `JDK`1.8以上
  2. `Spring Boot2.0`以上
### Subscriber Server安装
  1. 安装Canal Server。（本工具第一版本需要依赖Canal，后面版本会计划内置）Canal地址：https://github.com/alibaba/canal/wiki/QuickStart。
  2. clone本项目，找到**subscriber-server**项目，修改`application.yml`如下：
  ```java
    # canal server (单机客户端配置)
    canal:
      server:
        # canal server的ip
        ip: 127.0.0.1
        # canal server的port
        port: 11111
        # 默认订阅(当所有客户端均为订阅，将会使用此默认订阅)
        subscribe: db.table1,db.table2
  ```
  3. 打包**subscriber-server**并运行，默认端口`8889`
  4. 打包 Subscriber-client

### 在项目中使用
  1. 项目中加入上一步打包之后的依赖：
  ```xml
<dependency>
    <groupId>com.galen</groupId>
    <artifactId>subscriber-client</artifactId>
    <version>1.0.0-RELEASE</version>
    <scope>compile</scope>
</dependency>
```
  2. 在Spring Boot启动类上加入注解`@EnableSubscriber`
  3. 创建订阅类，实现`com.galen.subscriber.client.DataSync`接口，并加上`@Subscriber`注解，如：
  ```java
@Subscriber(db = "minifranchise_dev", table = "n_product_detail", name = "galen1", contextId = "galenTB")
@Slf4j
public class GalenTableSync implements DataSync {
    @Override
    public ACK doSync(Exchange exchange) {
        System.err.println(exchange.getTableName() + 1);
        System.err.println(JSON.toJSONString(exchange));
        return ACK.SUCCESS();
    }
}
```