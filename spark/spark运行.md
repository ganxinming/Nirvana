### spark-shell命令及其常用的参数如下：

```
./bin/spark-shell --master <master-url>
Spark的运行模式取决于传递给SparkContext的Master URL的值。Master URL可以是以下任一种形式：
* local 使用一个Worker线程本地化运行SPARK(完全不并行)
* local[*] 使用逻辑CPU个数数量的线程来本地化运行Spark
* local[K] 使用K个Worker线程本地化运行Spark（理想情况下，K应该根据运行机器的CPU核数设定）
* spark://HOST:PORT 连接到指定的Spark standalone master。默认端口是7077.
* yarn-client 以客户端模式连接YARN集群。集群的位置可以在HADOOP_CONF_DIR 环境变量中找到。
* yarn-cluster 以集群模式连接YARN集群。集群的位置可以在HADOOP_CONF_DIR 环境变量中找到。
* mesos://HOST:PORT 连接到指定的Mesos集群。默认接口是5050。

–master：这个参数表示当前的Spark Shell要连接到哪个master，如果是local[*]，就是使用本地模式启动spark-shell，其中，中括号内的星号表示需要使用几个CPU核心(core)；
–jars： 这个参数用于把相关的JAR包添加到CLASSPATH中；如果有多个jar包，可以使用逗号分隔符连接它们；

比如，要采用本地模式，在4个CPU核心上运行spark-shell：
cd /usr/local/spark
./bin/spark-shell --master local[4]

或者，可以在CLASSPATH中添加code.jar，命令如下：
cd /usr/local/spark
./bin/spark-shell --master local[4] --jars code.jar 

可以执行“spark-shell –help”命令，获取完整的选项列表，具体如下：
cd /usr/local/spark
./bin/spark-shell --help

该命令省略了参数，这时，系统默认是“bin/spark-shell –master local[*]”，也就是说，是采用本地模式运行，并且使用本地所有的CPU核心。
bin/spark-shell


最后，可以使用命令“:quit”退出Spark Shell，如下所示：
```





## spark使用jar包运行

```
1.编写好scala代码

2.打包，排除所有依赖，仅保留代码

3./usr/local/spark/bin/spark-submit --class com.spark.gan.wc.WordCount /usr/local/spark/myjar/spark-core.jar
```

