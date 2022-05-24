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