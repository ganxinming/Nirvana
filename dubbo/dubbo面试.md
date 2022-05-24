> dubbo中zookeeper做注册中心,如果注册中心集群都挂掉,那发布者和订阅者还能通信吗?



答案是可以的,为什么呢?我们看下面三个图,我们看到zookeeper的信息会缓存到本地作为一个缓存文件,并且转换成`properties`对象方便使用.



## dubbo默认序列化？java哪种合适

hessian2序列化：hessian是一种跨语言的高效二进制序列化方式。但这里实际不是原生的hessian2序列化，而是阿里修改过的，它是dubbo RPC默认启用的序列化方式。

hessian 是一个比较老的序列化实现了，而且它是跨语言的，所以不是单独针对java进行优化的。而dubbo RPC实际上完全是一种Java to Java的远程调用，其实没有必要采用跨语言的序列化方式（当然肯定也不排斥跨语言的序列化）。

现在有一些新的序列化：

专门针对Java语言的：Kryo，FST等等
跨语言的：Protostuff，ProtoBuf，Thrift，Avro，MsgPack等等
这些序列化方式的性能多数都显著优于 hessian2 （甚至包括尚未成熟的dubbo序列化）。所以我们可以 
为 dubbo 引入 Kryo 和 FST 这两种高效 Java 来优化 dubbo 的序列化。