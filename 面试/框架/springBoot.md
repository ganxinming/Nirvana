### @SpringBootApplication

@SpringBootApplication注释等同于同时使用@Configuration，@EnableAutoConfiguration和@ComponentScan及其默认属性。

### 1.@Configuration，@SpringBootConfiguration(声明一个配置类)

@Configuration这个注解的作用就是声明当前类是一个配置类，然后Spring会自动扫描到添加了@Configuration的类，读取其中的配置信息，而@SpringBootConfiguration是来声明当前类是SpringBoot应用的配置类，项目中只能有一个。所以一般我们无需自己添加。

#### 2.@EnableAutoConfiguration(开启自动配置，完成starter下的web，mvc自动配置)

开启自动配置，告诉SpringBoot基于所添加的依赖，去“猜测”你想要如何配置Spring。比如我们引入了spring-boot-starter-web，而这个启动器中帮我们添加了tomcat、SpringMVC的依赖，此时自动配置就知道你是要开发一个web应用，所以就帮你完成了web及SpringMVC的默认配置了！我们使用SpringBoot构建一个项目，只需要引入所需框架的依赖，配置就可以交给SpringBoot处理了。

### 重点：

```
内部主要依靠导入了AutoConfigurationImportSelector类
@Import(AutoConfigurationImportSelector.class)
```

而主要是通过：==SpringFactoriesLoader工具类 SpringFactoriesLoader.loadFactoryNames()==
把 spring-boot-autoconfigure.jar/META-INF/spring.factories中每一个xxxAutoConfiguration文件都加载到容器中，spring.factories文件里每一个xxxAutoConfiguration文件一般都会有下面的条件注解:

- @ConditionalOnClass ： classpath中存在该类时起效
- @ConditionalOnMissingClass ： classpath中不存在该类时起效
- @ConditionalOnBean ： DI容器中存在该类型Bean时起效，等等......

##### ==SpringFactoriesLoader==

SpringFactoriesLoader属于Spring框架私有的一种扩展方案(类似于Java的SPI方案java.util.ServiceLoader)，其主要功能就是从指定的配置文件META-INF/spring-factories加载配置，spring-factories是一个典型的java properties文件，只不过Key和Value都是Java类型的完整类名，比如：

```
# Auto Configure
org.springframework.boot.env.EnvironmentPostProcessor=\
  com.baomidou.mybatisplus.autoconfigure.SafetyEncryptProcessor
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.baomidou.mybatisplus.autoconfigure.MybatisPlusLanguageDriverAutoConfiguration,\
  com.baomidou.mybatisplus.autoconfigure.MybatisPlusAutoConfiguration
```

这也是为什么现在很少看到配置文件了，其实都是有默认配置在starter下的。

#### 3.@ComponentScan()

配置组件扫描的指令
提供了类似与<context:component-scan>标签的作用
通过basePackageClasses或者basePackages属性来指定要扫描的包。

# 面试题

### **什么是springboot ？有哪些特点**

1.简化开发配置，取代springxml配置。(但是保留了xml)

2.内嵌tomcat，无需部署war包

3.spring boot来简化spring应用开发，约定大于配置，去繁从简

4.自动配置spring添加对应功能starter自动化配置 

## **什么是 Spring Boot Stater ？**

启动器是一套方便的依赖没描述符，它可以放在自己的程序中。你可以一站式的获取你所需要的 Spring 和相关技术，而不需要依赖描述符的通过示例代码搜索和复制黏贴的负载。

例如，如果你想使用 Sping 和 JPA 访问数据库，只需要你的项目包含 spring-boot-starter-data-jpa 依赖项，你就可以完美进行。 

(其实就是一组该技术所需要的jar包，且进行了一些默认配置，解决了版本兼容的问题)

 ***\**\*Spring Boot中的监视器是什么？\*\**\***

Spring boot actuator是spring启动框架中的重要功能之一。Spring boot监视器可帮助您访问生产环境中正在运行的应用程序的当前状态。 

有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为HTTP URL访问的REST端点来检查状态。



### springboot配置文件优先级？

依次为： bootstrap.properties -> bootstrap.yml -> application.properties -> application.yml

其中 bootstrap.properties 配置为最高优先级

先加载的会被后加载的覆盖掉，所以.properties和.yml同时存在时，.properties会失效，.yml会起作用。

#### 两种类型

在 Spring Boot 中有两种上下文，一种是 bootstrap, 另外一种是 application, bootstrap 是应用程序的父上下文，也就是说 bootstrap 加载优先于 applicaton。

bootstrap 主要用于从额外的资源来加载配置信息，还可以在本地外部配置文件中解密属性。这两个上下文共用一个环境，它是任何Spring应用程序的外部属性的来源。bootstrap 里面的属性会优先加载，它们默认也不能被本地相同配置覆盖。

3、bootstrap/ application 的应用场
bootstrap.yml 和application.yml 都可以用来配置参数。

bootstrap.yml 可以理解成系统级别的一些参数配置，这些参数一般是不会变动的。
application 配置文件这个容易理解，pplication.yml 可以用来定义应用级别的，主要用于 Spring Boot 项目的自动化配置。

bootstrap 配置文件有以下几个应用场景：

使用 Spring Cloud Config 配置中心时，这时需要在 bootstrap 配置文件中添加连接到配置中心的配置属性来加载外部配置中心的配置信息；
一些固定的不能被覆盖的属性
一些加密/解密的场景；
