### java三个特性：

**封装**：尽可能地隐藏内部的细节，只保留一些对外接口使之与外部发生联系。比如：将变量private修饰，通过并生成get/set方法来进行访问。

**继承**：使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能。继承所描述的是“is-a”的关系。

1、子类拥有父类非private的属性和方法。

2、子类可以拥有自己属性和方法，即子类可以对父类进行扩展。

3、子类可以用自己的方式实现父类的方法。（重写）。

4、向上转型：是从一个叫专用类型向较通用类型转换，所以它总是安全的

5、向下转型：不安全，从通用类型转到专用类型，有失败的风险。**如果向下转型想要编译和运行都成功，必须使用强制转型语法，还必须要求运行起来父类引用确实指向该子类的对象，不然一个猫类型转成的父类型转成狗类型，不是很荒唐吗**。

**多态**：多态是同一个行为具有多个不同表现形式或形态的能力。

##### 多态存在的三个必要条件

- 继承
- 重写
- 父类引用指向子类对象：提高了代码的维护性和扩展性。向上转型 ，动态绑定（动态链接） 

**实现多态的技术称为**：

动态绑定（dynamic binding），是指在运行期间判断所引用对象的实际类型，根据其实际的类型调用其相应的方法。比如：方法参数是个父类型，传入子类型，调用方法会判断实际类型调用。

(1) invokevirtual指令中的#15指的是AutoCall类的常量池中第15个常量表的索引项。这个常量表(**CONSTATN_Methodref_info** ) 记录的是方法f1信息的符号引用(包括f1所在的类名，方法名和返回类型)。JVM会首先根据这个符号引用找到调用方法f1的类的全限定名: hr.test.Father。这是因为调用方法f1的类的对象father声明为Father类型。

(2) 在Father类型的方法表中查找方法f1，如果找到，则将方法f1在方法表中的索引项11(如上图)记录到AutoCall类的常量池中第15个常量表中(**常量池解析** )。这里有一点要注意：如果Father类型方法表中没有方法f1，那么即使Son类型中方法表有，编译的时候也通过不了。因为调用方法f1的类的对象father的声明为Father类型。

(3) 在调用invokevirtual指令前有一个aload_1指令，它会将开始创建在堆中的Son对象的引用压入操作数栈。然后invokevirtual指令会根据这个Son对象的引用首先找到堆中的Son对象，然后进一步找到Son对象所属类型的方法

Java中除了final，static，private和构造方法是静态绑定。(绑定：方法和类绑定)其他都是动态绑定，所以说，只有方法才可能是动态绑定，变量是静态绑定。

主要表现：

1.重写：发生在父子类中，方法名称相同，参数列表相同，方法体不同，遵循“运行期绑定”，看对象的类型的调用方法。

两同：方法名称相同，参数列表相同
两小：子类方法的返回值类型小于或等于父类的。void时，必须相等，基本类型时，必须相等
引用类型时，小于或等于，子类抛出的异常小于或等于父类的。
一大：子类方法的访问权限大于或等于父类的。

2.重载：在一个类中，方法名称相同，参数列表不同，方法体不同。遵循“编译期绑定”，看引用的类型来绑定方法

### 接口和抽象类区别：

1.接口中所有的方法隐含的都是抽象的，而抽象类则可以同时包含抽象和非抽象的方法。

2.接口中所有抽象方法必须实现，抽象类可以部分实现。

3.接口可以实现多个，抽象类只能实现继承一个。

4.接口中方法默认public abstract，而抽象类可以有其他访问修饰符和非abstract方法。

5.接口中变量默认是public final，抽象类默认是非final变量。

### 访问修饰符：

​			访问权限   类   包  子类  其他包

  　　　　  public     ∨   ∨    ∨     ∨          （对任何人都是可用的）

   　　　　 protect    ∨   ∨   ∨     ×　　　 （同包下，和继承的类可以访问）

   　　　　 default    ∨   ∨   ×     ×　　　 （包访问权限，即在整个包内均可被访问）

   　　　　 private    ∨   ×   ×     ×　　　 （除类型创建者和类型的内部方法之外的任何人都不能访问的

### java异常：

顶级异常：Throwable，Throwable又派生出Error类和Exception类。

Error：多为JVM错误，该错误不能被程序员通过代码处理。

Exception：检查异常（checked exception）：需要手动处理(try catch和throws)，和运行时异常(RuntimeException)运行时可能发生的异常，如分母为0，数组越界，空指针等。

### java泛型：

相当于一颗语法糖，在编译过程中，泛型会确保类型安全，正确检验泛型结果后，会将泛型的相关信息擦除（泛型擦除）。

好处：对于ArrayLIst使用泛型，可以免去转换类型，确保集合是同一类型。

**有两种限定通配符**：一种是<? extends T>它通过确保类型必须是T的子类来设定类型的上界，另一种是<? super T>它通过确保类型必须是T的父类来设定类型的下界

泛型类和泛型方法：

```
泛型类：是在实例化类的时候指明泛型的具体类型；
public class Pair<T> { //T is type parameters
	private T first;
	private T second;
	Pair<Integer> p=new Pair<Integer>()
	//初始化的时候一定要指定泛型
泛型方法：是在调用方法的时候指明泛型的具体类型 。
//在泛型类中声明了一个泛型方法，使用泛型E，这种泛型E可以为任意类型。可以类型与T相同，也可以不同。
//由于泛型方法在声明的时候会声明泛型<E>，因此即使在泛型类中并未声明泛型，编译器也能够正确识别泛型方法中识别的泛型。
        public <E> void show_3(E t){
            System.out.println(t.toString());
        }
```

#### java类型：

##### 基本类型：默认整型是int，默认的浮点是double。所以对于float要加1.1f。

- byte/8  boolean 1
- char/16 short/16
- int/32 float/32 
- long/64 double/64

包装类型：

```
Integer x = 2;     // 装箱  调用了 Integer.valueOf(2);如果数值在[-128,127]之间，便返回指向缓冲池中已经存在的对象的引用；否则创建一个新的Integer对象。
int y = x;         // 拆箱  调用了 Integer.intValue(x);
intValue()是把Integer对象类型变成int的基础数据类型；
parseInt()是把String 变成int的基础数据类型；
Valueof()是把给定的String参数转化成Integer对象类型；
```

##### 引用类型：

**String：**

String 被声明为 final，因此它不可被继承，不可重写，不可更改。(Integer等包装类也不能被继承)

在 Java 8 中，String 内部使用 char 数组存储数据。

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

##### 不可变好处：

**1.可以缓存 hash 值**

对于map的键值，不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

**2. String Pool 的需要**

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

**3.安全性**

线程可以安全访问，网络传输可以安全。

##### String Pool：

字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程中将字符串添加到 String Pool 中。

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

#### 常问方法：

#### 1.==和equals（）比较：

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否是同一引用地址，而 equals() 判断引用的对象内容是否等价。

##### 在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象散列值也相等。

##### 因为：equals如果不重写，默认还是==，如果重写了，里面是用hashcode来判断是否是同一个对象值。

##### hashcode必须重写，因为hashcode默认是取对象的地址，这样的话两个新new的对象永远不可能相等。

自反性：对于任意的对象x，x.equals(x)返回true(自己一定等于自己)；
对称性：对于任意的对象x和y，若x.equals(y)为true，则y.equals(x)亦为true；
传递性：对于任意的对象x、y和z，若x.equals(y)为true且y.equals(z)也为true，则x.equals(z)亦为true；
一致性：对于任意的对象x和y，x.equals(y)的第一次调用为true，那么x.equals(y)的第二次、第三次、第n次调用也均为true，前提条件是没有修改x也没有修改y。

对于非空引用x，x.equals(null)永远返回为false。

### 2.toString()：

默认返回 类名@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

### 3.clone():

clone() 是 Object 的 protected 方法，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。(需重写 如下：return (复制的类型名)super.clone();)

注意：1、Object类虽然有这个方法，但是这个方法是受保护的（被protected修饰），所以我们无法直接使用。2、使用clone方法的类必须实现Cloneable接口，否则会抛出异常CloneNotSupportedException。

浅拷贝：只用重写上面的colne方法就行。只进行值拷贝，所以对于基本类型只是复制了值，对于引用类型，只复制地址。地址还是指向对象，并且可以修改，所以这样不安全。

```java
implements Cloneable 
  /**
     *  重写clone()方法
     * @return
     */
  @Override
    public Object clone() {
        //浅拷贝
        try {
            // 直接调用父类的clone()方法
            return super.clone();
        } catch (CloneNotSupportedException e) {
            return null;
        }
    }
```

深拷贝：对象也复制一份。通过对象序列化实现深拷贝，默认会将该对象的整个对象图进行序列化，再通过反序列即可完美地实现深拷贝。

```
1.代码手动赋值
2.实现seriazilabe接口，通过流ObjectOutputStream进行序列化和反序列化，直接使用工具类	SerializationUtils.clone()就可以了
3.JSon序列化和反序列化(性能差)
```



### 关键字：

#### final：

##### 1.数据：声明数据为常量，不能被改变的常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

##### 2.方法：声明方法不能被子类重写。

##### 3.类：声明类不允许被继承。

#### static：

##### 1.静态变量：

- 静态变量：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。
- 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。

##### 2.静态方法：

在类加载的时候就存在了,只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字。

**3. 静态语句块**

静态语句块在类初始化时运行一次。

**4. 静态内部类**

非静态内部类依赖于外部类的实例，而静态内部类不需要。

#### 初始化顺序：

- 父类（静态变量、静态语句块）

- 子类（静态变量、静态语句块）

- 父类（实例变量、普通语句块）

- 父类（构造函数）

- 子类（实例变量、普通语句块）

- 子类（构造函数）

  当然了以上都是使用进行new初始化的时候。

  假如仅仅只是调用sout(B.a)时，即子类调用父类静态属性时，只会执行父类静态代码块，而不执行子类静态代码块。

  其实也很好理解，只调用了父类静态属性，所以父类进行“初始化”，而子类不进行初始化。(java中只有首次使用才会初始化，注意初始化是类的，执行构造方法是对象)

  特例：此类中并不会完成myParent初始化，因为在编译阶段就将final static 变量推送至“ 当前类” 常量池中了(即使将myParent编译后class删除，也无影响，因为已经放入常量池了)，可直接使用，但是需要注意的是，这个final变量的值在编译期是已知的，如果是未知的(如：使用uuid生成的值)，那还是得加载myparent初始化。(说明了final只将编译期已知的值推送至常量池，运行期的值不能)

  ```
  public class myTest1 {
      public static void main(String[] args) {
          System.out.println(myParent.str);
      }
  }
  
  class myParent {
      public static final String str = "常量的str";
      static {
          System.out.println("myParent的加载");
      }
  }
  
  ```

### 反射：

每个类都有一个 **Class** 对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。 

作用：在运行时获得某些未知的类。

##### 为啥JDBC使用class.forName("com.mysql.jdbc.Driver")?

在Java中如果想要使用一个类，则必须要求该类已经被加载到Jvm中，加载的过程实际上就是通过类的全限定名来获取定义该类二进制字节流，然后将这个字节流所表示的静态存储结构转换为方法去的动态运行时数据结构。同时在在内存中实例化一个java.lang.Class对象，作为方法区中该类的数据访问入口(供我们使用)。

##### class.forName作用就是：将类加载到JVM中，可是并没有初始化对象啊？且看他的代码

```
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    public Driver() throws SQLException {
    }
    static {
        try {
        //在静态代码块中，有初始化对象。
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

所以他被加载到JVM,然后执行了静态代码块。如果new一个对象，会相当于new了两次。

使用：Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

- **Field** ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的属性；
- **Method** ：得到Method method=clazz.getMethod();对象关联的方法；可以使用 method.invoke() 方法调用该对象方法。
- **Constructor** ：可以用 Constructor con =clazz.getConstructor();和con.newInstance() 创建新的对象。

优点：可扩展性强，增加程序的灵活性。

缺点：安全性不可靠，会暴露内部，效率低（因为是在已加载的代码基础上得到的）。

应用：jdbc，spring的IOC，代理模式，Cglib字节码增强。

### 设计模式：

面向对象设计原则

1.单一职责：改变类的只有一个原因，一个类只负责一个职责。

2.里式替换：所有引用子类的地方都能被父类替换。

3.开闭原则：对扩展开发，对修改关闭。

4.依赖倒转：依赖于抽象，不依赖于具体实现。针对接口编程，而不是针对实现编程

5.接口隔离：使用多个专门的接口，而不使用单一的总接口。

6.迪米特法则：类知道最少，对其他对象知道最少，不必发生直接的作用。

7.合成复用：使对象组合起来使用，而不是通过继承。

UML类图关系：泛化(继承，三角实线)，实现（三角虚线），聚合（可单独成立），组合（不可单独），依赖，关联。

***1.创建型：***
创建型模式主要用于描述如何创建对象
6种：
***单例模式\***，***\*工厂方法模式\****，***\*抽象工厂模式\****，**原型模式**，**建造者模式(适用于多构造器时，自定义构造对象)**
***2.结构型*****
2.结构型模式主要用于描述如何实现类或对象的组合
7种：
***适配器模式(tomcat连接器中，转换serveletRequest)\****，***\*桥接模式\****，***\*组合模式\****，***\*装饰模式\****，***\*外观模式\****，***\*享元模式\****，***\*代理模式。\****
***3.行为型***
行为型模式主要用于描述类或对象怎样交互以及怎样分配职责
11种：
职责链模式，***\*命令模式\****，解释器模式，***\*迭代模式\****，中介者模式，备忘录模式，***\*观察者 模式\****，***\*状态模式\****，***\*策略模式\****，***\*模板方法模式\****，访问者模式。

特别注意的设计模式：

##### 建造者模式：将一个复杂的对象分解为多个简单的对象，然后一步一步构建而成，但每一部分是可以灵活选择的。

##### 通过指挥者来构建具体设计一个类的具体构建(不同构建，不同方法)。

##### 适配器：将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作。适配器模式分为类结构型模式和对象结构型模式两种。

##### 桥接：将不同维度分离出来，使它们可以独立变化。

##### 装饰：指在不改变现有对象结构的情况下，动态地给该对象增加一些职责（即增加其额外功能）。使用组和替代继承关系。缺点：产生不同的装饰，这样就增加了很多子类。

##### 组合：它是一种将对象组合成树状的层次结构的模式，用来表示“部分-整体”的关系

##### 外观：是一种通过为多个复杂的子系统提供一个一致的接口，而使这些子系统更加容易被访问的模式

享元：运用共享技术来大量细粒度对象的复用。

##### 代理：由于某些原因需要给某对象提供一个代理以控制对该对象的访问。这时，访问对象不适合或者不能直接引用目标对象，代理对象作为访问对象和目标对象之间的中介。优点：

- 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用；
- 代理对象可以扩展目标对象的功能；
- 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度；

##### 责任链：当有请求发生时，可将请求沿着这条链传递，直到有对象处理它为止。客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递，所以职责链将请求的发送者和请求的处理者解耦了。

##### 策略：定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。

##### 模板方法：定义一个操作中的算法骨架，而将算法的一些步骤延迟到子类中，使得子类可以不改变该算法结构的情况下重定义该算法的某些特定步骤。它是一种类行为型模式。

##### 观察者：指多个对象间存在一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

#### 	BIO/NIO/AIO：

##### 同步和异步：当我们进行同步操作时，后续的任务是等待当前调用返回，才会进行下一步，异步则相反。

##### `阻塞`和`非阻塞`针对的是操作io后是否立即得到的返回结果而言的。阻塞：一直等待。非阻塞：立刻返回结果。

##### BIO:同步阻塞式IO,基于字节流，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如;socket通过在while循环中服务端会调用accept方法等待接收客户端的连接请求，一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成。 如果BIO要能够同时处理多个客户端请求，就必须使用多线程，即每次accept阻塞等待来自客户端请求，一旦受到连接请求就建立通信套接字同时开启一个新的线程来处理这个套接字的数据读写请求，然后立刻又继续accept等待其他客户端连接请求。执行while循环，当发现有人接入了accept，就开启一个新的线程，或者从线程池中取一个线程为他服务,如果没有线程了，则会一直等待或者遭到拒绝请求。

##### 主要用于：数据间的传输。分为管道流和节点流。

##### NIO:同步非阻塞 IO ，基于Channel、Selector、Buffer。关键是采用了Reactor事件驱动的思想来实现了一个多路转换器，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接 “有I/O请求” 时才启动一个线程进行处理。 NIO的最重要的地方是当一个连接创建后，不需要对应一个线程，这个连接会被注册到多路复用器上面，所以所有的连接只需要一个线程就可以搞定，当这个线程中的多路复用器进行轮询的时候，发现连接上有请求的话，才开启一个线程进行处理，也就是一个请求一个线程模式。

NIO与BIO最大的区别就是只需要开启一个线程就可以处理来自多个客户端的IO请求事件。

BIO是一个连接一个线程。

NIO是一个请求一个线程。

AIO是一个有效请求一个线程。

##### AIO:异步非阻塞式IO，异步 IO 操作基于事件和回调机制，应用操作直接返回，而不会阻塞在那里，当后台处理完成，操作系统会通知相应线程进行后续工作。

与NIO不同，当进行读写操作时，只须直接调用API的read或write方法即可。这两种方法均为异步的，对于读操作而言，当有流可读取时，操作系统会将可读的流传入read方法的缓冲区，并通知应用程序；对于写操作而言，当操作系统将write方法传递的流写入完毕时，操作系统主动通知应用程序。  即可以理解为，read/write方法都是异步的，完成后会主动调用回调函数。

- BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。
- NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。
- AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

#### socket和websocket区别：

WebSocket：同HTTP一样也是应用层的协议，但是它是一种**双向通信协议**，是建立在TCP之上的。Browser/Client Agent 都能主动的向对方发送或接收数据。

Socket：是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

#### netty：它对JDK中的NIO做了封装和优化，提供了更好的性能的同时，降低了使用的难度。





## java8特性

##### 1.为interface新增特性:可以有default方法,可以有static方法。

```
public interface Action {
    void shout();
 
    default void jump() {
        System.out.print("default jump()");
    }
 
    static void desc() {
        System.out.print("static desc()");
    }
}
a、default方法访问权限默认就是public（不填写时）default方法通过实现类对象调用，default方法支持实现类重写

b、static方法也是权限默认就是public（不填写时） static方法，只能通过interface名称来调用，static方法不支持实现类重写

c、private、protected均不能修饰defaut与static方法（跟interface原有规则保持一致，哈哈）
```

首次遇见是在sprinboot拦截器中:HandlerInterceptor。都是default，可以不用实现方法。

##### 2.lamba表达式是什么?

我们可以把它看成是一种闭包，它允许把函数当做参数来使用，是面向函数式编程的思想。

使用:() -> {} 就是这么简单使用。(仅当只有1个参数时，可以省略括号，其他情况都不行0,>1都不行)

那什么情况可以使用？假如接口有多个方法呢，他怎么知道实现哪个方法？

参数是一个接口时且里面只有一个方法时，默认就实现那个唯一的方法。

比如new Thread( () -> {} ); 其实我们都知道传入的是Runable接口，这时使用lamba表达式则默认是作为传入的参数实现那里面唯一的方法。

##### 2.简单使用：

```
//List类的forEach方法
items.forEach(c-> System.out.println(c));

//先使用过滤器
items.stream().filter(s->s.contains("B")).forEach(c1-> System.out.println(c1));

//Java8遍历map，这简直就是逆天操作
items.forEach((key,value)-> System.out.println("key:" + key + " value:" + value));

//将一组对象进行分组
Map<String, List<Person>> collect = personList.stream().collect(Collectors.groupingBy(c -> c.getAddress()));
        
```

##### 3.Stream使用:使用的方法按照下面的顺序调用：

```
stream+->filter+-> |sorted+-> |map+-> |collect|
```

stream:可以将集合数据转成流，而后面的那些操作就想管道对流进行操作。

1.foreach：遍历流数据。

2.filter:过滤出符合条件的数据。

3.limit:限制输出的数据量。

4.sorted:对流数据进行排序。

5.map:方法用于映射每个元素到对应的结果

6.collect:将结果集进行聚合，合成其他什么list都行。



# 枚举：

1.枚举类型使用

JDK1.5引入了新的类型——枚举。在 Java 中它虽然算个“小”功能，却给我的开发带来了“大”方便。

常用来表示那些可以明确范围的集合，比方说性别，季节，星期，月份等。(可以认为是一系列集合)

enum 和 class ，interface 的地位一样，使用enum定义。

1. ###### 枚举类的所有实例都必须放在第一行展示，不需使用 new 关键字，不需显式调用构造器。自动添加 public static final 修饰。

2. ###### 规定枚举类型的常量全部都是大写字母

3. ######  一个类所创建的对象个数是固定的

4. ###### 枚举类不能被继承，因为它本身就是继承自enum，java仅支持单继承。

5. 里面的枚举类型，其实每个都是对象。

6. 可以通过switch进行判断

```
public enum SWCodeEnum {
    CODE_OK_0(0, "Success"),
    CODE_OK(200, "OK"),
    CODE_FAIL(-1, "操作失败"),
    CODE_RPC_ERROR(-2, "远程调度失败"),
    CODE_UNKNOWN(-3, "未知错误"),
    CODE_REDIS_ERROR(-4, "存储redis错误"),
    CODE_CPBEAN_ERROR(-5, "复制bean失败"),
    CODE_PARAM_ERROR(460, "请求参数错误");

    private int code;
    private String msg;

    SWCodeEnum(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }

    public static SWCodeEnum codeOf(int code) {
        for (SWCodeEnum state : values()) {
            if (state.getCode() == code) {
                return state;
            }
        }
        return SWCodeEnum.CODE_UNKNOWN;
    }
}
```

Enum使用:

1.像普通类一样定义构造参数，定义成员变量。

2.在第一排就声明枚举类型名，声明默认使用构造方法，所以构造方法就是用来为这枚举设定初始值。

3.调用时，使用SWCodeEnum.CODE_OK_0(其实这样就相当于获得对象了，然后直接.方法就可以调用了)

(CODE_OK_0默认是static的，所以可以直接类名调用)



Enum和静态类的区别:

1.静态类定义的类型就只能是单个的，但是枚举类型，相当于是定义的多个集合对象。每个对象含有多个值。

2.限制值的范围，比如可能我们只想选取某个字符集合里的某个，而此时传入的参数是String，那么这个String就有可能取很多值了。如果传入的是枚举类型，那么他的值就只能限定在某个集合的范围内了。(因为枚举类型就那几个)

示例

```
public enum SWC{
    private SWCodeEnum enum;
    private int code;
    private String msg;
	//正因为传入的是枚举类型，所以值只能是集合里的，如果是String，你还得判断是否是合理的值
    SWC(int code, String msg,SWCodeEnum enum) {
        this.code = code;
        this.msg = msg;
        this.enum=enum;
    }
}
```

注意事项:

1.枚举不可继承其他类，但是可以实现接口。(jvm在生成枚举时已经继承了Enum类)

2.枚举类不可以被继承，在jvm在生成时，默认声明为final。

3.枚举类是单例的，即那些对象也都是单例的。

4.因为是单例的，所以不允许被克隆clone()方法。不允许反序列化，因为是单例。(枚举要保证永远是单例的)

5.当使用compareTo()比较枚举时，比较的是什么？

枚举类型的compareTo()方法比较的是枚举类对象的ordinal的值(序列号)。

6.当使用equals()比较枚举的时候，比较的是什么？

枚举类型的equals()方法比较的是枚举类对象的内存地址，作用与等号等价。

(即：equals和==作用一样，都是比内存地址，而compareTo比较的是里面序列号的值)

(枚举类中默认隐含两个字段 name(枚举类型名)  和  ordinal(序列号：即定义的顺序号) )



## 线程池：避免了创建线程和销毁线程的资源损耗。

Executors提供四种线程池：

- newCachedThreadPool ：缓存线程池，如果线程池长度超过处理需要，可回收空闲线程，若无可回收，则新建线程。
- newFixedThreadPool ： 定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
- newScheduledThreadPool ： 计划线程池，支持定时及周期性任务执行。
- newSingleThreadExecutor ：单线程线程池，用唯一的线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

先了解下线程池必备的以下几个属性：

poolSize : 当前程序运行的线程数。

coreSize:  核心线程数。

MaximumPoolSize: 线程池中最大线程数。

keepAliveTime：线程空闲时间超过，则会超时退出。

BlockingQueue ：设置一个阻塞用来存放将要执行的等待任务。

(线程安全：保证取队列元素时一个只能取一个，且头元素没有时不能取，满时不能加入)

### 线程池的运行流程：

当需要执行一个任务时：

poolsize < coresize ->开启一个新的线程执行

poolsize = coresize && 队列未满  -> 这时已经达到了核心线程数，线程不够了，加入到阻塞队列中，慢慢执行。

Poolsize < MaximumPoolSize && 队列已满  -> 这时队列都满了，那不行啊 ，快爆了，在开启一个新线程吧。

（通常超出核心线程的线程是“借”的，也就是说超出核心线程的情况算是一种能够预见的异常情况，并且这种情况并不常常发生（如果常常发生，那我想你应该调整你的核心线程数了）

Poolsize = MaximumPoolSize  && 队列满了 -> 已经所有办法用尽了，会根据饱和策略RejectedExecutionHandler拒绝新的任务。

(如果设置了keepAliveTime，可以通过以下配置来确定小于corePoolSize时，是否启动超时处理。

但是对于超出MaximumPoolSize，一定会进行超时处理，因为这些线程本就不是正常情况。

**allowCoreThreadTimeOut：**是否允许核心线程超时退出。

如果该值为false(默认)，且poolSize<=corePoolSize，线程池都会保证这些核心线程处于存活状态，不会超时退出。(所以只要不是超过corePoolSize，线程都会空闲而不是销毁)

如果为true，则不论poolSize的大小，都允许超时退出。

如果poolSize>corePoolSize，则该参数不论true还是false，都允许超时退出。

)

### 下面就看看创建线程的源码：

(所以大概流程：执行一个任务，如果线程少于coresize则启动新线程，如果等于了coresize，则加入队列等待。如果队列满了，实再不行了，则在开启新线程。如果线程等于最大线程了，执行饱和策略拒绝任务)

```
自定义线程：可以发现，其实就是我们说的那几个参数，特别简单。
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) 
                          
/**
*coresieze和maximumPool相同，所以这也是为什么他是固定大小。
并且阻塞LinkedBlockingQueue没有初始化大小，是个无界队列。所以进来的任务，线程不够就会都加入队列等待。
keepAliveTime：0，默认超过最大线程才会启用超时，意味着：一超过最大线程数，立即关闭。
*/
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    
/**
* coresieze和maximumPool都为1
*/
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
    
/**
* 核心线程为0，最大线程数无限大。意味着，所有线程都有keepalivetime，都有一小段的生命周期。(缓存线程也就是说，执行的线程会缓存一段时间，然后到时间释放)在使用这类无限大的线程池时，非常容易内存消耗殆尽。这也是为什么阿里不建议使用Executors创建线程池。其实自定义线程池非常好用，而且自己看了就会非常明确。
SynchronousQueue（同步队列）一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作(即每个任务都必须有线程接管，否则不加入)，否则插入操作一直处于阻塞状态
*/
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
    
/**
*定时线程使用的是ThreadPoolExecutor子类
*/
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }

public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
        return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1));
    }

使用executorService.execute(Runnable)或executorService.submit(Callable)
```

(线程是线程Thread，任务是任务Runnable。

创建一个Runnable，只是实现了run方法,这是任务，不是线程。他并不能执行，所以需要传入Thread中，然后执行。

启动线程的唯一方法就是通过Thread类的start()实例方法。

)

## 线程池拒绝策略：

当线程池满了，队列也满了，这时候需要执行拒绝策略。默认策略感觉不太好，如果一满就会报异常停止运行。

##### AbortPolicy(默认)

默认策略，直接跑出异常阻止系统正常运行

##### CallerRunsPolicy

“调用者运行”一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将任务回馈至调用方，比如main线程

##### DiscardOldestPolicy

抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务

##### DiscardPolicy

直接丢弃任务，不给予任何处理也不跑出异常，如果允许任务丢失



## runable和callable

runable: 使用execute，无返回值

callable:使用submit，有返回值，返回的是future。

(注意如果在任务就获得值，future.get(),将会阻塞主线程，导致线程串行处理。一般的做法是在任务中将所有future用list收集起来，出来线程池的处理，在future.get())



### 三：重写流程

# 常用类底层原理：

## String

## 数组

## List

## Map

##### 初始化map容量

当我们通过HashMap(int initialCapacity)设置初始容量的时候，HashMap并不一定会直接采用我们传入的数值，而是经过计算，得到一个新值，目的是提高hash的效率。(1->1、3->4、7->8、9->16)

找到比传入的指定值大的第一个2的幂
