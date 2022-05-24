# 1.什么是BIO

# BIO：是阻塞IO

核心：面向流（stream）

顶级抽象类：InputStream,OutputStream,Reader,Writer.（所有流的底层都是面向字节的，字符流只是因为更好的操作。）

### 正因为是抽象类，不是接口，注定了一个类要么是输入流，要么是输出流，不可能两个都是。因为只能继承一个类啊，如果是接口的话可能就不同了。与NIO很大区别。

基本流程：围绕一个while循环

输入流:                                                          输出流：

1.open a stream						1.open a stream

2.while  more information				2.while  more information

3.read  information					3.write information

4.close information					4.close information

流分类：

节点流：从特定的地方读写的流，例如磁盘，内存（比如：FileInputstream）

过滤流：过滤流是基于一个已存在的输入输出流运行。（对其他流就行包装的，Buffer）

IO流的链接：

流和流之间（比如，buffer）可以嵌套使用，像连在一起。



# 什么是NIO？

### 它有三个核心概念，Selector,Channel,Buffer。

#### 核心：面向块(block)或缓冲区（buffer），buffer底层本就是一块内存，他实际上就是一个数组，数据的读写都是通过buffer实现。

### 传统的BIO，就是建立一个socket，在某个端口监听连接

执行while循环，当发现有人接入了accept，就开启一个新的线程，或者从线程池中取一个线程为他服务。

注意：1.线程的数量是有限的，当达到万级并发时，这样的模型肯定是不行的，因为线程都不够用

​	2.这个8899端口只是用来监听连接的，实际上传送数据时，都是开启一个新的端口，如12001等。

### 而NIO相反，用一个线程监听所有连接。

八大类型都有各自对应的buffer类型，如：intBuffer,charBuffer等

Channel：指的是既可以写入数据，又可以读取数据的对象。

类似于Stream，但又不一样，它既可以读又可以写，双向的。它必须搭配buffer一起使用，不存在直接在Channel直接读和取数据。数据先存在到buffer中（相当于数据存放的中转站），然后channel可以读。

每次需要读写转换时需要调用方法 byteBuffer.flip();

## flip源码如下：

```
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

Buffer是特定基本类型元素的线性有限序列。除内容外，Buffer区的基本属性还包括capacity(容量)、limit(限制)和position(位置)：

capacity是它所包含的元素的数量。缓冲区的容量不能为负并且不能更改。(即容量)
limit是第一个不应该读取或写入的元素的索引。缓冲区的限制不能为负，并且不能大于其容量。（即每次需求容量的位置+!,比如写入10个，他就在10. 例如只读7个，他就在7,<位置从0开始>）

position是下一个将要读取或写入的元素的索引（总是比已有数据的位置多1）。缓冲区的位置不能为负，并且不能大于其限制(即当前位置)

将数据读入到buffer开始：position从0开始，capacity和limit在最容量最大地方。

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20171126170354033?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzA5NjA4OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 假如读入了5个，将position置0

执行完flip：将limit=position，此时想要把数据写出，limit是最大位置，所以positon不会超过这个位置

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20171126170617503?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzA5NjA4OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# Buffer：重要四个属性capacity(容量)、limit(限制)和position(位置)，mark（标记）

capacity(容量)，limit(第一次在capacity，以后都在最大实际容量的前一位)，position（下一个将要读取的位置）

疑问：1.执行两次flip：执行两次作用都是一样，将limit=positon,position=0;

​	   2.无论是读还是写，底层都是对数组进行操作。（写：给数组赋值。读：从数组取值）

​	   3.执行flip，仅仅只是改变position和limit的位置，buffer里面存的值实际都存在，并没有改动。

执行个实验：

一：同一个buffer，两个不同channel对其操作。

两次flip。

```java
FileInputStream in=new FileInputStream("1.txt");
 //通过inputstream获得输入流
 FileChannel fileChannel=in.getChannel();
 //设置buffer的长度
 ByteBuffer byteBuffer=ByteBuffer.allocate(215);
 //将数据读入buffer
 fileChannel.read(byteBuffer);

 byteBuffer.flip();
 //循环读出
 byte[] bytes=new byte[215];
 int i=0;
 while (byteBuffer.remaining()>0){
     bytes[i]=byteBuffer.get();
     i++;
 }
 System.out.println(new String(bytes));
 System.out.println("########");
 in=new FileInputStream("2.txt");
 fileChannel=in.getChannel();
// byteBuffer.clear();
 byteBuffer.flip();
 fileChannel.read(byteBuffer);
 byteBuffer.flip();
 //循环读出
  bytes=new byte[215];
  i=0;
 while (byteBuffer.remaining()>0){
     bytes[i]=byteBuffer.get();
     i++;
 }
 System.out.println(new String(bytes));
```

结论：第一次buffer的limit在读入数据后，limit的位置就已经确定了，而flip并不能将limit复原到capacity。

第二次写入数据到buffer，也只能写到limit的位置，后面的都忽略了。那么如果想要写入所有数据怎么办？

原则上，我们只需要将limit复原到capacity就行，clear()就是这个作用（复原）。

buffer里面还有更多对这三个元素进行操作的方法。

mark：将当前位置做个标记，有相应方法将postition复原到mark，而不是0位置,那么就可以从mark处进行操作。

#### 结论：0<=mark<=position<=limit<=capacity



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



## Protocol Buffers

为什么要用Protocol Buffers？用于网络数据传输。

网络数据传输可行的几种方式：

1.使用java序列化。（缺点只能是java语言）

2.使用特定的编码，将数据进行编码传输，到达后进行解码。（适用小数据，速度快）

3.使用xml数据进行传输。（太复杂，看的压力有点大）

### Protocol Buffers的优点：灵活高效，适用多种语言

### 使用步骤：

1.定义.proto file

2.use protocol buffer complier

3.Use java protocol buffer API to read and write message.



proto format :

```protobuf
syntax = "proto2";

package tutorial;
//如果不定义java包，默认使用package，作为标识，防止类的冲突
option java_package = "com.example.tutorial";
//定义一个class的名字，如果不定义，默认是proto的文件名
option java_outer_classname = "AddressBookProtos";

message Person {
//required，表示必须赋值，否则就不能初始化，将会抛出运行时异常。
  required string name = 1;
  required int32 id = 2;
//optional，表示可以设值或者不设值。各种类型有默认值。
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }
//repeated,表示可重复，可以当做一个数组或者list.
  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```



不能继承，只能组合使用



### Thrift(RPC框架)

主要用于各个服务RPC之间的通信，支持跨语言，在多种语言之间调用。

Thrift 是个典型的CS结构，服务端和客户端可以使用不同语言开发。但是如果使用了不同语言，那么一定有个中间语言来关联两种语言，这种语言就是IDL（接口描述语言，即.thrift文件）。

Thrift ：1.不支持无符号类型。（因为他要满足所有语言的特点，比如java，不支持无符号）

支持的类型：（选自官网）

### 数据类型：

##### 基本类型：

bool: A boolean value (true or false)
byte: An 8-bit signed(有符号) integer
i16: A 16-bit signed integer
i32: A 32-bit signed integer
i64: A 64-bit signed integer
double: A 64-bit floating point number
string: A text string encoded using UTF-8 encodin

##### 特殊类型：

Structs ：相当于类（protobuf中message差不多）

Containers：容器类，里面含有set，list，map等

Exceptions：异常

Services ：定义一系列方法（类似接口,这些方法就是我们远程调用的方法）



### Thrift使用

定义一个thrift文件，由thrift文件生成双方的接口，model，在生成的model及接口中会有解码编码的代码。

对应下面的代码设置。

对应下面代码：

阻塞式IO：来一个请求，起一个线程

非阻塞IO：传输格式是frame，其实在NIO中，也用到lframe格式

对应下面代码：

一个Thrift的构成，主要就是由上面的构成，数据传输格式，传输方式，服务类型组成.



# 编码的发展史

ASCII码(美国人自己用) 占7位,能表示127位字符.

iso-8859-1 (增加了一位) 占8位，一个字节 ,能表示256个字符.

但是还是不能满足我们，因为就中国汉字就有几千

出现了

gb2312(国标) 能表示大部分字，但是还有少数不能表示。每个字符占2个字节

gbk 在上面的基础上加了更多字

big5 （台湾省的繁体字）

这时候美国说：这样下去不行啊，你搞一个，他搞一个编码。

于是出现了unicode。

Unicode 采用两个字节表示一个字符,能表示世界上所有语言。

美国人不热议了，我们国家本来可以用一个字节表示，为啥要用两个?

这时出现了utf

UTF 是一种存储方式，Unicode是一种编码方式。

UTF-8（**变长字节表示形式**）,一般来说UTF-8通过三个字节表示汉字.

