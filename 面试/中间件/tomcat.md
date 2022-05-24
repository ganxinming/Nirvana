#### 1.为什么出现tomcat？

主要有server,service,context,host,engine,wrapper,connector这7个容器来组装。

tomcat本质上是一个由java编写的servlet程序。(必须在java环境中跑)。

仔细想想，如果没有tomcat仔细想想，他们和java怎么通信的？当然是通过http请求，去访问本地的资源。那么可想而知，他做的事情有：

1.tomcat本质上就是个大容器，装载了很多资源供其他人访问。保证资源能顺利的被URL访问到。

2.定义统一接口供外界访问，被调度的请求应该走什么样的业务逻辑，保证请求调用正确的servlet。(如果只用java，这些进程间的通信，都需要你去实现)

3.管理servelt，并且tomcat的开关就能保证资源是否开放。

4.处理多线程连接，保证多个线程接入正确访问。

#### 2.tomcat启动

因为也是个java文件，所以必须找到main方法入口。即在**bootstrap.java**中启动

#### 3.tomcat结构

1.连接器(Coyote)：将Socket请求包装成servletRequest，将servletResponse解析成响应返回。

2.容器(catalina)：我们的逻辑程序。

**一个连接器对应一个容器，一个容器可对应多个连接器，一个连接器-容器称为service（其实在源码里看到，这是个数组，进行初始化）。**



### 连接器(Coyote):

封装了底层网络通信协议，只负责具体的协议和IO的操作。只是负责封装request，response。这两个并不能直接和servlet通信。由Catalina转换成servletRequest才能通信。

### 容器(Catalina):

server.xml中包含了Catalina的所有配置，具体可以去看。重点提下下面的service标签， 一个service由连接器和容器构成。可以看到标签就跟下面图一样。

**(总结：server.xml包含了所有配置，而里面的service标签里配置了连接器和容器**

Connector标签：连接器

Engine标签：容器catalina

**)**

#### IO模型与协议：

支持IO：NIO（默认）,AIO/NIO2,APR(是c/c++编写的库，需要本地安装)。**(BIO在8.5之后就被移除了)**

支持协议：HTTP1.1(默认)，	AJP	,	HTTP2.0。



由图可知一个server就是一个tomcat，他是整个tomcat项目。

一个server可以有多个service，一个service包含多个连接器(Connector)和一个容器(Container).

连接器作用:

Connector的底层是使用socket进行连接，Request和Response时按照http协议（协议可选）来封装，所以Connector同时实现了TCP/IP和HTTP协议.

Connector(多个)主要作用：负责处理浏览器发送过来的请求，将其转化成request和response对象(servlet能识别的对象)，然后产生一个线程用于处理该请求，具体是进入到容器(Container底层也是个servlet容器)中进行处理。

容器作用:

Container(一个)容器是子容器的父接口，所有的子容器都继承该接口。其下的四个子容器接口为：Engine、Host、Context、Wrapper。(其中配置文件只配置了Engine、Host，其他两个是非必须的)

Engine：接受Connector传来的request和response对象，选择合适的host进行处理。(包含多个host)

Host:	代表一个虚拟主机，也是一个站点,即配置域名和webapps文件夹的地方。(包含多个Context)

Context：代表一个url访问应用，过域名可以直接访问到ROOT根目录里面的应用就是主应用。(包含多个Wrapper)

Wrapper:   每一个Wrapper封装一个servlet.

#### 总结：大概的结构就是上面了。



对于一个web容器而言，简而言之，它是系统的一个守护进程，守护着对这台服务器某个端口发起的请求，基于这一点，它就需要一个监听程序，这个监听程序来获取来自这个端口的特定请求的数据，ok，直接点讲，我们这里使用Socket来获取某个端口，通常是80端口的http请求，通过简单的Java程序的死循环（粗糙的做法，后面逐步优化）来实现不断的获取80端口http请求，来达到监听80端口http请求的目的。

**注意tomcat针对的是socket请求。**

EndPoint：将socket请求转发给Processor转化成http请求。

Processor：确保每次出来的都是http请求。(所以ProtocolHandler，其实是保证能处理socket连接转成http连接)

Adapter:适配器当然就是将request适配成servletRequest。(适配器模式)

### 流程：

点击starup.bat(查看该批处理文件，会发现加载bootstrap类),加载bootstrap.java完成一系列配置文件的加载。如：serverl.xml啊，加载各个组件完成初始化init()。并开始start()监听端口，准备接受客户端请求。



### service.xml配置如下：可使用jconsole工具 查看线程使用情况。



#### 启动流程：

一：调用bootstrap类启动(其实startup做的事情也就是调用bootstrap类)

bootstrap做的事情：

1.初始化ClassLoader类加载器(没有他无法加载后面代码)

2.类加载生成catlina实例，并通过反射调用catlina三个方法。

具体:通过init()方法进行initClassLoader()进行初始化类加载器。得到类加载器后，通过反射调用catlina的setAwait(true)，load(args)，start();这三个方法。	

所以总的来说：bootstrap主要完成了init()方法，调用catlina的setAwait(true)，load(args)，start()四个方法。

二：Catlina的启动(因为上面调用了start)

顺序调用setAwait(true)，load(args)，start()方法。

setAwait(true)：设置一个标记，为后面的start()使用。当通过后面的start()方法启动完服务器后，会检查这个值为true还是false，如果为true，调用Catlina的await()方法，tomcat继续开着。如果为false，则调用Catlina的stop()方法，关闭tomcat。（反正就是通过设置这个参数，来判断是启动还是关闭）

load(args)：创建Digester去解析server.xml，并根据server的配置文件创建server实例。(加载配置文件就是这里)

然后调用server实例的init()方法，进行server的初始化。Server初始化时，又会调用其内部的Service的init方法。

start():主要调用了server的start()方法来启动服务器，然后server的start()方法再调用多个service的start()方法。

（其实后面所有的初始化都是在load()读取了配置文件后进行初始化了）

三:Server的启动(之前catlina调用了server的init和start方法)

一个server含有多个service。service也是个接口，默认实现类是StandardService，所以我们后面谈论的service都是StandardService。每一个service都是一个对外提供服务的组件，每一个Service中都包含：一个Container+多个Connector，所以service中的方法，都是用来操作这两种类的。

四:之后就是service下一系列的初始化。比如:service再调用Connector的init方法创建并初始化.如下图:

![1575524926863](../../../../../home/badribbit/.config/Typora/typora-user-images/1575524926863.png)

五：Lifecycle接口

这里有必要提下Lifecycle(生命周期)接口，他的实现类是LifecycleBean类。就上面提到的service和server都继承自LifecycleBean，都含有生命周期。实际上凡是拥有“生命周期”的组件都实现了Lifecycle接口，Lifecycle接口的作用就是进行“生命周期的规定”。

内部主要内容:

1.定义了13个String常量，这些常量被用于定义“LifecycleEvent事件”的type属性中 。这样设计的好处是，只需要发送一种类型事件LifecycleEvent，而根据里面type属性的值的不同，来判定是什么事件。（否则13种事件就得定义13中事件类）。

2.定义了三个方法用于管理监听器LifecycleListener。分别是增加，查找，删除监听器。

3.定义了四个生命周期方法：init(),start(),stop(),destory()

LifecycleBean正是对上面方法进行了实现。

六:Connector内部构成：

Connector的主要任务是负责处理浏览器发送过来的请求，并创建一个Request和Response的对象用于和浏览器交换数据，然后产生一个线程用于处理请求，Connector会把Request和Response对象传递给该线程，该线程的具体的处理过程是Container容器的事了。

Connector中使用ProtocolHandler来具体处理请求，不同的ProtocolHandler代表不同协议的请求类型。

ProtocolHandler中又包含三个重要的组件：

Endpoint，用于处理底层socket网络连接。

Processor，用于将socket接收到的socket封装成request。

Adapter，用于将requst(转成servletReqest)交给Container进行处理。(采用适配器，和servlet容器适配)

(在Connector进行初始化init时，主要就是用来初始化ProtocolHandler)

Connector的初始化：在standServie(即service)中有个startInternal()方法,里面包含了Connector.start()方法。

在他的start()方法中又有startInternal()对ProtocolHandler进行初始化。

七:tomcat优化

1.内存空间优化:默认空间很小，当项目很大时，会将空间占满，抛出异常。修改启动时JVM参数。

2.连接器优化:

优化参数配置:最大线程数，最大空闲线程，连接超时等

```
<Connector port="8080" 
protocol="org.apache.coyote.http11.Http11NioProtocol" //配置方式
        redirectPort="8443"
        URIEncoding="UTF-8"
        minSpareThreads="100"
        maxSpareThreads="500"
        maxThreads="1000"
        acceptCount="700"
        connectionTimeout="30000"
        enableLookups="false"/>
        
```

3.线程池优化:

Executor代表了一个线程池，可以在Tomcat组件之间共享。使用线程池的好处在于减少了创建销毁线程的相关消耗，而且可以提高线程的使用效率。

配置完，则tomcat所有组件可以共享，则大家的线程被统一管理。(使用时记得配置到Connector连接器)

```
 <Executor name="tomcatThreadPool"   
         namePrefix="catalina-exec-"   
         maxThreads="1000"   
         minSpareThreads="100"  
         maxIdleTime="60000"  
         maxQueueSize="Integer.MAX_VALUE"  
         prestartminSpareThreads="false"  
         threadPriority="5"  
         className="org.apache.catalina.core.StandardThreadExecutor"/>  
```

4.IO优化:处理连接请求，BIO/AIO/APR(需要安装APR库，从操作系统级别解决异步IO问题，质的飞跃，就是安装比较麻烦)的选择。一般情况就选择BIO，如果可以，选择APR库再好不过了，大幅提升性能。

