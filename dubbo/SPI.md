### Dubbo SPI 示例（增强javaSPI）

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



#### @Adaptive 适配器注解

作用：根据参数进行适配不同的实现类。本身不具备任何作用

比如：A接口，B实现类，C实现类，在接口上使用@Adaptive接口，会生成一个适配器类，具体逻辑就是根据url取选择不同的实现类

```
@SPI(value = "hessian2", scope = ExtensionScope.FRAMEWORK)
public interface Serialization {
		@Adaptive
    ObjectInput deserialize(URL url, InputStream input) throws IOException;

}

public class Hessian2Serialization implements Serialization

public class JavaSerialization implements Serialization


```



## 区别：

1.java的spi会将实现的类全部初始化

2.dubbo是根据key进行明确加载类，减少资源浪费





curl  http://triggerAppealTask

```
suspendAppealByOrderFinalStateTask
```