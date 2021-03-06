# JMM(java内存模型)

　**概念：**Java内存模型是一种**抽象的概念，并不真实存在**，定义了**Java程序在各种平台下对内存访问的机制及规范**。通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。

通过这组规则控制程序中各个变量在共享数据区域和私有数据区域的访问方式，JMM是围绕原子性，有序性、可见性展开的。



#### 工作内存

由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存(有些地方称为栈空间)，用于存储线程私有的数据。

主要存储当前方法的所有本地变量信息(工作内存中存储着主内存中的变量副本拷贝)，每个线程只能访问自己的工作内存，即线程中的本地变量对其它线程是不可见的，就算是两个线程执行的是同一段代码，它们也会各自在自己的工作内存中创建属于当前线程的本地变量，当然也包括了字节码行号指示器、相关Native方法的信息。注意由于工作内存是每个线程的私有数据，线程间无法相互访问工作内存，

==该线程的工作区间会从主内存中拷贝一份数据放到工作内存中，供该线程做执行，执行完操作之后，cpu会将工作内存中的数据刷新到主内存中去==

因此存储在工作内存的数据不存在线程安全问题。

#### 主内存

主要存储的是Java实例对象，所有线程创建的实例对象都存放在主内存中，不管该**实例对象是成员变量还是方法中的本地变量(也称局部变量)**，当然也包括了共享的类信息、常量、静态变量。由于是共享数据区域，多条线程对同一个变量进行访问可能会发现线程安全问题。

Java内存模型中规定所有变量都存储在主内存。主内存是共享内存区域，所有线程都可以访问。

#### 读取变量流程

- 线程对变量的操作(读取赋值等)必须在工作内存中进行
- 首先要将变量从主内存拷贝的自己的工作内存空间
- 然后对变量进行操作，操作完成后再将变量写回主内存(所以可能发生线程安全问题)
- 不能直接操作主内存中的变量，工作内存中存储着主内存中的变量副本拷贝

<img src="../../../Library/Application Support/typora-user-images/image-20220522225617209.png" alt="image-20220522225617209" style="zoom:33%;" />



### 数据内存分配

- 根据虚拟机规范，对于一个实例对象中的成员方法而言，如果方法中包含本地变量是基本数据类型（boolean,byte,short,char,int,long,float,double），将直接存储在工作内存的帧栈结构中。
- 若本地变量是引用类型，那么该变量的引用会存储在工作内存的帧栈中
- 对象实例将存储在主内存(共享数据区域，堆)中，但对于实例对象的成员变量，不管它是基本数据类型或者包装类型(Integer、Double等)还是引用类型，都会被存储到堆区
- 至于static变量以及类本身相关信息将会存储在主内存中
- 倘若两个线程同时调用了同一个对象的同一个方法，那么两条线程会将要操作的数据拷贝一份到自己的工作内存中，执行完成操作后才刷新到主内存

<img src="../../../Library/Application Support/typora-user-images/image-20220522230249175.png" alt="image-20220522230249175" style="zoom:33%;" />

### 存在的线程安全问题

A工作内存进行操作，写入主内存，此时B可能读到X=1或者X=2，取决于什么时候读，所以这就是线程安全问题

<img src="../../../Library/Application Support/typora-user-images/image-20220522230734180.png" alt="image-20220522230734180" style="zoom:33%;" />

为了解决类似上述的问题，JVM定义了一组规则，通过这组规则来决定一个线程对共享变量的写入何时对另一个线程可见，这组规则也称为Java内存模型（即JMM），JMM是围绕着程序执行的原子性、有序性、可见性展开的，下面我们看看这三个特性。



### 数据同步的八大原子操作

- **lock(锁定)：**把一个**变量标记为一条线程独占状态**
- **unlock(解锁)**：把一个**处于锁定状态的变量释放**出来，释放后的变量才可以被其他线程锁定
- **read(读取)：**把一个**变量值从主内存传输到线程的工作内存**中，以便随后的load动作使用
- **load(载入)：**把read操作从主内存中得到的**变量值放入工作内存的变量副本**中
- **use(使用)：**把**工作内存中的一个变量值传递给执行引擎**
- **assign(赋值)：**把一个从执行引擎接收到的**值赋给工作内存的变量**
- **store(存储)：**把**工作内存中的一个变量的值传送到主内存**中，以便随后的write的操作
- **write(写入)：**把store操作从工作内存中的一个**变量的值传送到主内存的变量**中

<img src="../../../Library/Application Support/typora-user-images/image-20220523091405528.png" alt="image-20220523091405528" style="zoom:33%;" />

<img src="../../../Library/Application Support/typora-user-images/image-20220523091502872.png" alt="image-20220523091502872" style="zoom:33%;" />

## 并发编程的可见性，原子性与有序性问题

#### 原子性

　　**定义：**原子性指的是一个操作是**不可中断的，不可分割的**，即使是在多线程环境下，一个操作一旦开始就不会被其他线程影响。

#### 可见性

　　**定义：**可见性指的是**当一个线程修改了某个共享变量的值，其他线程是否能够马上得知这个修改的值**。

#### 有序性(多线程才会指令重排)

　　**定义：**程序执行的顺序，按照代码的先后顺序执行。

　　对于单线程的执行代码，我们总是认为代码的执行是按顺序依次执行的，但对于多线程环境，则可能出现乱序现象【指令重排导致】，重排后的指令与原指令的顺序未必一致。



## JMM如何解决原子性、可见性、有序性问题

### 原子性问题(加锁解决)

　　除了JVM自身提供的对基本数据类型读写操作的原子性外，可以通过**synchronized**和**Lock**实现原子性。**【synchronized和Lock能够保证任一时刻只有一个线程访问该代码块】**

### 可见性问题(加锁和volatile解决)

- volatile关键字可以保证可见性。

  当一个共享变量被volatile修饰时，它会保证修改的值立即被其他的线程看到，即修改的值立即更新到主存中，当其他线程需要读取时，它会去内存中读取新值。

- synchronized和Lock也可以保证可见性。

  因为它们可以保证任一时刻只有一个线程能访问共享资源，并在其释放锁之前将修改的变量刷新到内存中。

### 有序性问题(加锁和volatile解决)

- 可以通过**synchronized**和**Lock**来保证有序性。

　　synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。

- volatile关键字禁止指令重排。(通过加一层内存屏障)

## happen-before 规则

1. 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作==可见==，而且第一个操作的执行顺序排在第二个操作之前。
2. 两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行（个人感觉**解释成“生效可见于” 更准确**）。如果重排序之后的执行结果与按照happens-before关系来执行的结果一致，那么这种重排序并不非法。(可以指令重排)

happens-before关系保证正确同步的多线程程序的执行结果不被改变。

==HB 原则是对单线程环境下的指令重排序以及多线程环境下的线程间数据的可见性进行的约束。==

在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系，这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。



## **为什么需要happens-before**

**JVM会对代码进行编译优化，会出现指令重排序情况**，为了避免编译优化对并发编程安全性的影响，**需要happens-before规则定义一些规则，满足规则则可以编译优化的场景**，**都是为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度。**

### HB 有哪些规则？

这个大家都非常熟悉了应该，大部分书籍和文章都会介绍，这里稍微回顾一下：

1. 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作，即在一个线程内必须保证语义串行性（准确地讲是控制流顺序而不是代码顺序）
2. 锁定规则：锁(unlock)操作必然发生在后续的同一个锁的加锁(lock)之前，也就是说，在监视器锁上的解锁操作必须在同一个监视器上的加锁操作之前执行。
3. **volatile变量规则**：对一个变量的写操作先行发生于后面对这个变量的读操作； volatile变量的写，先发生于读，这保证了volatile变量的可见性，简单的理解就是，volatile变量在每次被线程访问时，都强迫从主内存中读该变量的值，而当该变量发生变化时，又会强迫将最新的值刷新到主内存，任何时刻，不同的线程总是能够看到该变量的最新值。
4. **传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C；**
5. 线程启动规则：Thread对象的start()方法先行发生于此线程的每一个动作；
6. 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生；
7. 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行；
8. 对象终结规则：对象的构造函数执行和结束(即整个构造过程)，先于finalize()方法





# 指令重排序（因为需要提升多线程下执行效率）



#### 为了提高性能，编译器和处理器常常会对指令重排，一般分为以下三种：

==编译器优化的重排== ：编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序；

==指令并行的重排==  ：现代处理器采用了指令级并行技术来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序；

==内存系统的重排==：由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行的。



### as-if-serial语义(保证重排序后不影响最终结果，这就是为啥能重排的原因，因为不会影响结果)

  不管怎么重排序【编译器和处理器为了提高并行度】，（单线程）程序的执行结果不能被改变。

所有的动作都可以为了优化而被重排序，==但是必须保证它们重排序后的结果和程序代码本身的应有结果是一致的==。Java编译器、运行时和处理器都会保证**单线程**下的as-if-serial语义。

## as-if-serial和happens-before区别

- as-if-serial语义保证==单线程内程序的执行结果不被改变==，happens-before关系保证正==确同步的多线程程序的执行结果不被改变。==
- as-if-serial语义给编写单线程程序的程序员创造了一个幻境：单线程程序是按程序的顺序来执行的。happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻境：正确同步的多线程程序是按happens-before指定的顺序来执行的。
- as-if-serial语义和happens-before这么做的目的，都是为了在不改变程序执行结果的前提下，==尽可能地提高程序执行的并行度==。

## 内存屏障

**内存屏障的作用：**

- 防止指令之间的重排序
- 强制把写缓冲区/高速缓存中的脏数据等写回主内存，让缓存中相应的数据失效

**硬件层的内存屏障分为两种：Load Barrier 和 Store Barrier即读屏障和写屏障**

- 对于Load Barrier来说，在指令前插入Load Barrier，可以让高速缓存中的数据失效，强制从新从主内存加载数据
- 对于Store Barrier来说，在指令后插入Store Barrier，能让写入缓存中的最新数据更新写入主内存，让其他线程可见

**java内存屏障：**

- LoadLoad屏障：对于这样的语句Load1; LoadLoad; Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕
- StoreStore屏障：对于这样的语句Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见
- LoadStore屏障：对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕
- StoreLoad屏障：对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的。在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能

**final语义中的内存屏障：**

- 新建对象过程中，构造体中对final域的初始化写入和这个对象赋值给其他引用变量，这两个操作不能重排序
- 初次读包含final域的对象引用和读取这个final域，这两个操作不能重排序（先赋值引用，再调用final值）



# 总结

总而言之，我们应该清楚知道，

- ==JMM就是一组规则==，==这组规则规定了对内存访问的机制及规范，意在解决在并发编程可能出现的线程安全问题==

- 并提供了==内置解决方案（happen-before原则，as-if-serial规则）==及其==外部可使用的同步手段(synchronized/volatile等)==
- 确保了程序执行在多线程环境中的应有的==原子性，可见性及其有序性==。
- 解决在并发环境下由于 CPU缓存、编译器和处理器的指令重排序 导致的原子性，可见性、有序性问题