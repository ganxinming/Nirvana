# MAC安装

```
1.进入ssh的目录 
cd ~/.ssh
2.一直回车就回看到id_rsa 和id_rsa.pub
ssh-keygen
3.ssh免密登录
cat id_rsa.pub >> authorized_keys

4.安装hadoop
brew install hadoop

5.安装成功
hadoop version

进入hadoop的目录：
cd /usr/local/Cellar/hadoop/3.3.1/libexec/etc/hadoop
④ 修改core-site.xml：

<configuration>
        <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:8020</value>
  </property>
 
  <!--用来指定hadoop运行时产生文件的存放目录  自己创建-->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>file:/usr/local/Cellar/hadoop/tmp</value>
  </property>
</configuration>

⑤ 修改hdfs-site.xml，配置namenode和datanode：

<configuration>
        <property>
                <name>dfs.replication</name>
                <value>1</value>
        </property>
        <!--不是root用户也可以写文件到hdfs-->
        <property>
                <name>dfs.permissions</name>
                <value>false</value>    <!--关闭防火墙-->
        </property>
        <!--把路径换成本地的name坐在位置-->
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/usr/local/Cellar/hadoop/tmp/dfs/name</value>
        </property>
        <!--在本地新建一个存放hadoop数据的文件夹，然后将路径在这里配置一下-->
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/usr/local/Cellar/hadoop/tmp/dfs/data</value>
        </property>
</configuration>
⑥ 修改 mapred-site.xml：

<configuration>
  <property>
    <!--指定mapreduce运行在yarn上-->
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
     <name>mapred.job.tracker</name>
     <value>localhost:9010</value>
  </property>
  <!-- 新添加 -->
  <!-- 下面的路径就是你hadoop distribution directory -->
  <property>
     <name>yarn.app.mapreduce.am.env</name>
     <value>HADOOP_MAPRED_HOME=/usr/local/Cellar/hadoop/3.3.1/libexec</value>
  </property>
  <property>
     <name>mapreduce.map.env</name>
     <value>HADOOP_MAPRED_HOME=/usr/local/Cellar/hadoop/3.3.1/libexec</value>
  </property>
  <property>
     <name>mapreduce.reduce.env</name>
     <value>HADOOP_MAPRED_HOME=/usr/local/Cellar/hadoop/3.3.1/libexec</value>
</property>
 
</configuration>

⑦ 修改yarn-site.xml:

<configuration>
	<property>
	    <name>yarn.nodemanager.aux-services</name>
	    <value>mapreduce_shuffle</value>
	  </property>
	<property>
	    <name>yarn.resourcemanager.address</name>
	    <value>localhost:9000</value>
	</property> 
	<property>
	  <name>yarn.scheduler.capacity.maximum-am-resource-percent</name>
	  <value>100</value>
	</property>
</configuration>


8.修改环境变量
vim ~/.bash_profile
export HADOOP_HOME=/opt/homebrew/Cellar/hadoop/3.3.1/libexec
export HADOOP_COMMON_HOME=$HADOOP_HOME
export PATH=$PATH:$HADOOP_HOME/bin:
9.source ~/.bash_profile刷新

10.cd /usr/local/Cellar/hadoop/3.3.1/bin
进入到hadoop的bin目录下执行./hdfs namenode -format

11.启动hadoop
cd /usr/local/Cellar/hadoop/3.3.1/libexec/sbin
./start-dfs.sh

浏览器中输入http://localhost:9870/，出现以下界面就说明成功了：

12.启动yarn
cd /usr/local/Cellar/hadoop/3.3.1/libexec/sbin
./start-yarn.sh
浏览器中打开http://localhost:8088/就会出现下图的界面:
```

/usr/local/spark/bin/spark-submit --class com.spark.gan.wc.WordCount /usr/local/spark/myjar/spark-core.jar