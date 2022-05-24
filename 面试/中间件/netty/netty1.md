Protocol Buffers:类似于一个数据格式（xml）更小，更快，更简单.

格式如下：

```
message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;
}
required ：代表该属性必须，没有报错
optional ：代表可有可无
后面的就是数据类型和属性名。 后面的数字不是指，而是属性的顺序。
```

stub(客户端)

skeleton(服务端)

rmi （远程方法调用,跨机器的，方法不在本台机器，仅仅局限于java代码

思考：为什么能远程调用方法？client调用远程方法，发信号给stub，stub告诉skeleton，skeleton告诉server。其中他们之间传递的方法是字节码进行序列化传输，得到序列化代码再进行反序列化得到代码。

）

RPC框架（远程服务调用，跨语言的，多种代码）

webSocket：在传统的http消息发送中，是必须发送request body等消息，但是两个消息交互，肯定没有那么大的数据量，每次这样传，非常浪费时间。

websocket允许只发送信息部分，支持全双工，服务端和客户端都可以实时交互。

### netty作用：

1.作为RPC框架实现远程调用

2.基于WebSocket，作为一个长连接的服务器，支持双向通信。

<font color="red">webSocket为什么是长连接呢？就传统的HTTP协议来说，他是短的协议，是无状态的协议，两次http请求你无法辨认是否是同一个人，所以产生了cookie和session。</font>

而socket就是维持一个长连接。

3.作为web容器，tomcat等等。



