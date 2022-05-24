# 注册一个dubbo服务

### 1.新建接口及实现类

### 2.新建 **provider.xml** 文件，对外发布服务

```
    <dubbo:application name="demo-provider"/>
  <!-- <dubbo:registry address="zookeeper://115.159.202.204:2181"/> -->
    <!--点对点的方式-->
    <dubbo:registry address="N/A" />
   <!--可设置多个协议-->
   <dubbo:protocol name="dubbo" port="20890"/>
<!--   <dubbo:protocol name="rmi" port="20890"/>-->

   <dubbo:service interface="com.springDubbo.dubboService.TestService" ref="testService" version="1.0.0"/>
```

### 3.服务实现类还需被spring管理，使用@Service

-Ddubbo.application.qos.enable=true -Ddubbo.application.qos.port=33333 -Ddubbo.application.qos.accept.foreign.ip=false



dubbo暴露一个服务，那消费者是怎么调用到服务呢？其实本质上dubbo就是个url，消费者通过调用url，访问资源

如下：协议://ip端口/服务?参数对(其实就是配置的一些)

```java
dubbo://192.168.234.1:20880/com.sihai.dubbo.provider.service.ProviderService?anyhost=true&application=provider&bean.name=com.sihai.dubbo.provider.service.ProviderService&bind.ip=192.168.234.1&bind.port=20880&dubbo=2.0.2&generic=false&interface=com.sihai.dubbo.provider.service.ProviderService&methods=SayHello&owner=sihai&pid=8412&qos.accept.foreign.ip=false&qos.enable=true&qos.port=55555&side=provider&timestamp=1562077289380
```

### 消费者

如果是注册在zk，可以直接调用

```java
<dubbo:reference id="providerService"
                     interface="com.sihai.dubbo.provider.service.ProviderService"/>
```

如果是，点对点，需要使用url进行dubbo直连

```java
<!--点对点方式-->
    <dubbo:reference id="providerService"
                     interface="com.sihai.dubbo.provider.service.ProviderService"
                     url="dubbo://192.168.234.1:20880/com.sihai.dubbo.provider.service.ProviderService"/>
```

