[TOC]



# Synchronized

## 加锁代价：

java的线程是映射到操作系统原生线程之上的，如果要阻塞或者唤醒一个线程就需要操作系统介入，==需要在用户态和内核态之间切换==，这种切换会消耗大量的系统资源，因为用户态和内核态都有各自专用的内存空间，专用的寄存器等，用户态切换至内核态需要传递给许多变量、参数给内核，内核也需要保护好用户态在切换时的一些寄存器、变量等，以便内核态调用结束后切换回用户态继续工作。

在Java早期版本中，synchronized属于重量级锁，效率低下，因为监视器（monitor）是依赖底层的操作系统的MutexLock来实现的，挂起线程和回复线程都需要转入内核态去完成，阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态切换需要耗费处理器时间，如果同步代码块中内容过于简单，这种切换的时间可能比用户代码执行的时间还长，时间成本相对较高，这也是为什么早期的synchronized效率低的原因。

在Java早期版本中，synchronized属于重量级锁，效率低下，因为监视器（monitor）是依赖底层的操作系统的MutexLock来实现的，挂起线程和回复线程都需要转入内核态去完成，阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态切换需要耗费处理器时间，如果同步代码块中内容过于简单，这种切换的时间可能比用户代码执行的时间还长，时间成本相对较高，这也是为什么早期的synchronized效率低的原因。

Java6之后，为了减少获得锁和释放锁所带来的性能消耗，引入了轻量级锁和偏向锁。

### 总结：

1.需要内核态和用户态切换，耗cpu



## 什么是锁呢？

每一个Java对象自打娘胎里出来就带了一把看不见的锁，它叫做内部所或者Monitor锁。Monitor的本质是依赖底层操作系统的==MutexLock==实现，操作系统实现线程之间的切换需要从用户态到内核态的转换，成本非常高。

**MutexLock**

Monitor是在JVM底层实现的，底层代码是C++。本质是依赖于底层操作系统的MutecLock实现，操作系统实现线程之间的切换需要从用户态到内核态的转换，状态转换需要耗费很多的处理器时间成本非常高，所以synchronized是Java语言中的一个重量级操作。



## 锁和线程怎么关联呢？

Monitor于Java对象以及线程是如何关联的。

1.如果一个Java对象被某个线程所著，==则该Java对象的Mark Word字段中LockWord指向monitor的起始地址==

2.Monitor的Owner字段回存放拥有相关联对象锁的线程ID



## 锁优化过程

### 偏向锁

当一段同步代码一直被同一个线程多次访问，由于只有一个线程那么该线程在后续访问时便会自动获得锁

多线程的情况下，锁不仅不存在多线程竞争，还存在锁由同一线程多次获得的情况，==偏向锁就是在这种情况下出现的==

#### 获取锁过程：

假如有一个线程执行到synchronized代码块的时候，JVM使用CAS操作把线程指针ID记录到==Mark Word==当中，并修改标偏向标示，标示当前线程就获得该锁。锁对象变成偏向锁（通过CAS修改对象头里的锁标志位），字面意思是“偏向于第一个获得它的线程”的锁。执行完同步代码块后，线程并不会主动释放偏向锁。

这时线程获得了锁，可以执行同步代码块。当该线程第二次达到同步代码块时会判断此时持有锁的线程是否还是自己（持有锁的线程ID也在对象头里），JVM通过account对象的Mark Word判断：当前线程ID还在，说明还持有着这个对象的锁，就可以继续进入临界区工作。==由于之前没有释放锁==，这里也就不需要重新加锁。如果自始至终使用锁的线程只有一个，很明显偏向锁几乎没有额外的开销，性能极高。

#### 偏向锁的撤销

偏向锁使用一种等到竞争出现才释放锁的机制，==只有当其他线程竞争锁时==，持有偏向锁的原来线程才会被撤销。

撤销需要等待全局安全点(该时间点上没有字节码正在执行)，同时检查持有偏向锁的线程是否还在执行： 

#### 偏向锁升级过程

第一个线程正在执行synchronized方法(处于同步块)，它还没有执行完，其它线程来抢夺，该偏向锁会被取消掉并出现锁升级。

此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程会进入自旋等待获得该轻量级锁。

(有竞争，升级轻量级锁，其他竞争线程自旋)

2.第一个线程执行完成synchronized方法(退出同步块)，则将对象头设置成无锁状态并撤销偏向锁，重新偏向 。

(当前线程执行完，释放锁，让其他线程去争夺偏向锁，如果得到偏向锁线程，又有竞争，又升级轻量级锁)

#### 作用

它的出现是为了解决只有一个线程执行同步时提高性能

参数说明：

偏向锁在JDK1.6以上默认开启，开启后程序启动几秒后才会被激活，可以使用JVM参数来关闭延迟 -XX:BiasedLockingStartupDelay=0

#### 总结：

1.偏向锁无需进行内核态切换，记录下当前线程ID到锁对象头里。

2.运行中，不会自动释放锁，除非有竞争。



## 轻量级锁(自旋+CAS)

有线程来参与锁的竞争，含锁线程由偏向锁，升级成轻量级锁。竞争线程进行自旋，本质就是自旋锁。

#### 作用

轻量级锁是为了在线程近乎交替执行同步块时提高性能。在没有多线程竞争的前提下，通过CAS减少重量级锁使用操作系统互斥量产生的性能消耗。

==(本质竞争偏向锁，竞争成功自己加偏向锁，竞争失败通过自旋，不断进行CAS获取轻量级锁)==

### 获取锁过程

假如线程A已经拿到了锁，这时线程B又来抢该对象的锁，由于该对象的锁已经被线程A拿到，当前锁已是偏向锁。而线程B在争抢时发现对象头MarkWord中的线程ID不是线程B自己的线程ID（而是线程A），那线程B就会进行CAS操作希望能获得锁。

此时线程B操作中有两种情况：

**如果锁获取成功**，直接替换MarkWord中的线程ID为B自己的ID（A→B），重新偏向于其他线程（即将偏向锁交给其他线程，相当于当前线程"被"释放了锁），该锁会保持偏向锁状态，A线程Over，B线程上位。

**如果锁获取失败**，则偏向锁升级为轻量级锁，此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程B会进入自旋等待获得该轻量级锁

JDK6之前 默认启用，默认情况下自旋的次数是 10 次 -XX:PreBlockSpin=10来修改或者自旋线程超过CPU核数一半。



#### 轻量级锁撤销：

轻量级锁么此退出同步块才释放锁



#### 轻量级锁和偏向锁的区别：

轻量级锁么此退出同步块都需要释放锁，而偏向锁时在竞争发生时才释放锁。



## 重量级锁

重量级锁 有大量的线程参与锁的竞争，冲突性很高 

有大量线程竞争则会升级重量级锁。

#### 为啥要升级？

想想轻量级锁，他是通过自旋一段时间，来换取CPU内核态转换。假如这种自旋的线程多了，还不如来一次内核态转换效率更高。

#### 作用：

所有线程阻塞，都在等待，不会销毁CPU，但是会很慢。





| 锁       | 优点                                                         | 缺点                                          | 适用场景                               |
| -------- | ------------------------------------------------------------ | --------------------------------------------- | -------------------------------------- |
| 偏向锁   | 加锁和解锁不需要额外的消耗，执行非同步方法相比仅存在纳秒级的差距 | 如果线程存在竞争，会带来额外的锁撤销的消耗    | 适用于只有一个线程访问同步代码块的场景 |
| 轻量级锁 | 竞争的线程不会阻塞，提高了程序的响应速度                     | 如果始终得不到锁竞争的线程，使用自旋会消耗CPU | 追求响应时间同步代码块执行速度非常快   |
| 重量级锁 | 线程竞争不适用自旋，不会消耗CPU                              | 线程阻塞，响应时间缓慢                        | 追求吞吐量同步代码块执行时间较长       |



## **Synchronized锁升级的过程总结：先偏向在自旋+cas，不行再阻塞**





## 对象的组成

Java对象保存在内存中时，由以下三部分组成：

> 1，对象头
>
> 2，实例数据(对象的实例数据就是在java代码中能看到的属性和他们的值)
>
> 3，对齐填充字节(把对象的大小补齐至8bit的倍数，没有特别的功能。) 

#### 一、对象头

对象头由以下三部分组成：

> 1，Mark Word
>
> <img src="../../Library/Application Support/typora-user-images/image-20220209193609241.png" alt="image-20220209193609241" style="zoom: 50%;" />
>
> 2，指向类的指针（对象指向它的类的[元数据](https://so.csdn.net/so/search?q=元数据&spm=1001.2101.3001.7020)的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。）
>
> 3，数组长度（只有数组对象保存了这部分数据。该数据在32位和64位JVM中长度都是32bit。）





curl http://localhost:8021/risk-abnormal-order-identify/manualTrigger/triggerScalpingOrderTask?triggerDay=1&currentShardItem=1&allShardItems=0