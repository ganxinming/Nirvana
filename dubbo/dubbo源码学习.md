## 1.dubbo中的任意接口实现，都可以统一成一个url

<img src="../../Library/Application Support/typora-user-images/image-20220426222754915.png" alt="image-20220426222754915" style="zoom: 25%;" />

可以被org.apache.dubbo.common.URL类表示

<img src="../../Library/Application Support/typora-user-images/image-20220426223527433.png" alt="image-20220426223527433" style="zoom: 25%;" />

#### 好处：

1.形成统一规范，代码易读



2.dubbo服务者像zk注册信息，调用doRegister方法，将信息封装到zk节点上

org.apache.dubbo.registry.zookeeper.ZookeeperRegistry#doRegister



3.dubbo消费者去zk订阅服务者信息，

