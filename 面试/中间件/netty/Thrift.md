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



![1556334772957](C:\Users\12714\AppData\Roaming\Typora\typora-user-images\1556334772957.png)

![1556335130938](C:\Users\12714\AppData\Roaming\Typora\typora-user-images\1556335130938.png)

对应下面的代码设置。

![1556335158457](C:\Users\12714\AppData\Roaming\Typora\typora-user-images\1556335158457.png)

![1556335243992](C:\Users\12714\AppData\Roaming\Typora\typora-user-images\1556335243992.png)

对应下面代码：

![1556335392977](C:\Users\12714\AppData\Roaming\Typora\typora-user-images\1556335392977.png)

![1556335458473](C:\Users\12714\AppData\Roaming\Typora\typora-user-images\1556335458473.png)

阻塞式IO：来一个请求，起一个线程

非阻塞IO：传输格式是frame，其实在NIO中，也用到lframe格式

对应下面代码：

![1556335488914](C:\Users\12714\AppData\Roaming\Typora\typora-user-images\1556335488914.png)



一个Thrift的构成，主要就是由上面的构成，数据传输格式，传输方式，服务类型组成.