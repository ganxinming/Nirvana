## SPI

SPI 全称为 Service Provider Interface，是一种服务发现机制。

API：给消费者调用。SPI：给服务者提供接口，用户可以自定义类实现接口。类似于留了个插件槽，用户可以自定义插件使用。

SPI：破坏了双亲委派机制。例如：利用spi加载jdbc的**java.sql.Driver**，保证各个sql厂家实现自己的类。SLFJ等，兼容各种日志。

##### 为啥加载数据库驱动需要SPI？

比如 java.sql.DriverManager/java.sq.Driver 等是 jre 核心内部类，在 $JAVA_HOME/jre/lib/rt.jar 中，==由 bootstrap classloader 负责加载他们；而 jdbc 实现类如 oracle.jdbc.driver.OracleDriver 是在程序运行时指定的类加载路径下的某个 jar 包中（类加载路径是通过环境变量 classpath 动态指定的），由 application class loader 负责加载他们==；所以当 bootstrap classloader 加载的 java.sql.DriverManager， 需要调用 application classloader 加载的 java.sq.Driver 的具体实现类 oracle.jdbc.driver.OracleDriver 时，由于类的可见性 （子类加载器加载的类对父类加载器加载的类默认不可见），按照双亲委派模型，这是不 work 的；





Bootstrap Classloader(只拿些rt.jar下的Stirng,Math等类)加载器拿到了Application ClassLoader加载器应该加载的类，就打破了双亲委派模型。(原本先让父类进行加载，父类本来也没有，才能被子类加载，现在通过SPI，直接在Bootstrap Classloader加载到JVM，这有个问题，如何在父加载器加载的类中，去调用子加载器去加载类？jdk提供了两种方式，Thread.currentThread().getContextClassLoader()和ClassLoader.getSystemClassLoader()一般都指向AppClassLoader，他们能加载classpath中的类。

其次线程的 contextClassLoader 是从父线程那里继承过来的，所谓父线程就是创建了当前线程的线程。程序启动时的 main 线程的 contextClassLoader 就是 AppClassLoader。这意味着如果没有人工去设置，那么所有的线程的 contextClassLoader 都是 AppClassLoader。




SPI则用Thread.currentThread().getContextClassLoader()来加载实现类，实现在核心包里的基础类调用用户代码)

(说白了在父加载器通过调用子加载器加载。)

SPI的接口是`Java核心库`的一部分，由Bootstrap类加载器加载的，而SPI实现的Java类一般是由系统类加载器加载的。引导类加载器（Bootstrap）是无法找到SPI的实现类的，因为它只加载Java的核心库。

它也不能委派给系统类加载器，因为它是系统类加载器的祖先类加载器。也就是说，类加载器的双亲委派模型无法解决这个问题。

为了解决这个问题，Java设计团队只好引入了一个不太优雅的设计：**线程上下文类加载器（Thread Context ClassLoader）**

### 双亲委派模式的破坏者：线程上下文类加载器

#### JDBC为什么要破坏双亲委派模型

> 因为类加载器收到加载范围的限制，在某些情况下父类加载器无法加载到需要的文件，这时候就需要委托子类加载器去加载class文件。

JDBC的Driver接口定义在JDK中，其实现由各个数据库的服务商来提供，比如MySQL驱动包。DriverManager类中要加载各个实现了Driver接口的类，然后进行管理，但是DriverManager位于$JAVA_HOME中jre/lib/rt.java包，由BootStrap类加载器加载，而其Driver接口的实现类是位于服务商提供的jar包，

> ==根据类加载机制，当被状态的类引用了另外一个类的时候，虚拟机就会使用装载第一个类的类装载器装载被引用的类。==

这就是说BootStrap类加载器还要去加载jar包中的Driver接口的实现类。
我们知道，BootStrap类加载器默认只负责加载$JAVA_HOME中的jre/lib/rt.jar里的所有的class，所以需要由子类加载器去加载Driver实现，这就破坏了双亲委派模型。



工作机制：创建流程

```java
public interface Robot {
    void sayHello();
}
```

接下来定义两个实现类，分别为 OptimusPrime 和 Bumblebee。

```java
public class OptimusPrime implements Robot {
    
    @Override
    public void sayHello() {
        System.out.println("Hello, I am Optimus Prime.");
    }
}

public class Bumblebee implements Robot {

    @Override
    public void sayHello() {
        System.out.println("Hello, I am Bumblebee.");
    }
}
```

接下来 META-INF/services 文件夹下创建一个文件，名称为 Robot 的全限定名 org.apache.spi.Robot。文件内容为实现类的全限定的类名，如下：

```
org.apache.spi.OptimusPrime
org.apache.spi.Bumblebee
```

做好所需的准备工作，接下来编写代码进行测试。

```java
public class JavaSPITest {

    @Test
    public void sayHello() throws Exception {
        ServiceLoader<Robot> serviceLoader = ServiceLoader.load(Robot.class);//相当于获取到所有实现接口的类
        System.out.println("Java SPI");
        serviceLoader.forEach(Robot::sayHello);
    }
}
```

最后来看一下测试结果，如下：

![img](http://dubbo.apache.org/docs/zh-cn/source_code_guide/sources/images/java-spi-result.jpg)

从测试结果可以看出，我们的两个实现类被成功的加载，并输出了相应的内容。关于 Java SPI 的演示先到这里，接下来演示 Dubbo SPI。

#### 例子：jdbc的实现,mysql包下会配置一个文件，包含需要SPI加载的类。

DriverManager.getcollection(url，user，password);

可以看到DriverManager在初始化时会使用ServiceLoader来加载java.sql.Driver的实现类。(加载驱动不应该是Class.forName吗？)

确实JDBC驱动的加载是在Class.forName这一步完成的，但是后面的jdbc版本也支持SPI加载。(即使不写Class.forName也能加载成功)

(判断逻辑：如果Class.forName加载了数据库驱动，则不适用SPI加载，否则选择SPI进行加载。根据url的不同加载不同的驱动)

这个自动加载采用的技术叫做SPI，数据库驱动厂商也都做了更新。可以看一下jar包里面的META-INF/services目录，里面有一个java.sql.Driver的文件，文件里面包含了驱动的全路径名。

比如mysql-connector里面的内容：

```css
com.mysql.jdbc.Driver
com.mysql.fabric.jdbc.FabricMySQLDriver
```



### 2.2 Dubbo SPI 示例（增强javaSPI）

Dubbo 并未使用 Java SPI，而是重新实现了一套功能更强的 SPI 机制。Dubbo SPI 的相关逻辑被封装在了 ExtensionLoader 类中，通过 ExtensionLoader，我们可以加载指定的实现类。Dubbo SPI 所需的配置文件需放置在 META-INF/dubbo 路径下，配置内容如下。

```
optimusPrime = org.apache.spi.OptimusPrime
bumblebee = org.apache.spi.Bumblebee
```

与 Java SPI 实现类配置不同，Dubbo SPI 是通过键值对的方式进行配置，这样我们可以按需加载指定的实现类。另外，在测试 Dubbo SPI 时，需要在 **Robot 接口上标注 @SPI** 注解。下面来演示 Dubbo SPI 的用法：

```java
public class DubboSPITest {

    @Test
    public void sayHello() throws Exception {
        ExtensionLoader<Robot> extensionLoader = 
            ExtensionLoader.getExtensionLoader(Robot.class);
        Robot optimusPrime = extensionLoader.getExtension("optimusPrime");//可以获得明确的实现类
        optimusPrime.sayHello();
        Robot bumblebee = extensionLoader.getExtension("bumblebee");
        bumblebee.sayHello();
    }
}
```

dubbo按照spi文件的用途，分成了三类目录

- META-INF/services 该目录下文件兼容jdk的spi
- META-INF/dubbo 该目录存放用户自定义实现
- META-INF/dubbo/internal 用于dubbo内部使用的spi



### @SPI注解使用

```
@SPI(value = "dubbo", scope = ExtensionScope.FRAMEWORK)
public interface Protocol {

    /**
     * Get default port when user doesn't config the port.
     *
     * @return default port
     */
    int getDefaultPort();
    
//默认以dubbo作为key
dubbo=org.apache.dubbo.rpc.protocol.ProtocolListenerWrapper
```

## 区别：

1.java的spi会将实现的类全部初始化

2.dubbo是根据key进行明确加载类，减少资源浪费

