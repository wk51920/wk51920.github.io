---
layout: post
title: Mac系统下，Hadoop 2.6.2 + ZooKeeper 3.4.6 + HBase 1.1.5 完全分布式环境搭建
---

# 简介

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HBase是Apache Hadoop的数据库，能够对大数据提供随机、实时的读写访问功能，具有开源、分布式、可扩展及面向列存储的特点。HBase是由Chang等人基于Google的Bigtable开发而成的。HBase的目标是存储并处理大型的数据，更具体来说是只需要使用普通的硬件配置即可处理由成千上万的行和列组成的大数据。

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HBase是一个开源的、分布式的、多版本的、面向列的存储模型。它可以直接使用本地文件系统，也可以使用Hadoop的HDFS文件存储系统。不过，为了提高数据的可靠性和系统的健壮性，并且发挥HBase处理大数据的能力，使用HDFS作为文件存储系统才更为稳妥。

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上内容来自《Hadoop实战》

# 配置环境

> + mac OSX EI Capitan 10.11.5 (master节点)
> + 虚拟机：Parallels Desktop
> + ubuntukylin 14.04 64bit * 2 (node1节点，node2节点)
> + Hadoop 2.6.2
> + ZooKeeper 3.4.6
> + HBase 1.1.5
> + 集群环境：HBaster Master: master(10.211.55.2)   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HRegion Server: node1(10.211.55.6) , node2(10.211.55.7)


# Hadoop环境配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于本系统使用完全分布式配置，并且文件系统选择使用HDFS，因此需要先搭建Hadoop环境。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这部分内容请移步本人另一篇博文，有详细的介绍：[Hadoop环境配置](http://blog.csdn.net/wk51920/article/details/51686038)。

# ZooKeeper环境配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ZooKeeper在完全分布式环境中必不可少。它的主要作用如下：

> + 存储HBase中ROOT表和META表的位置。
> + 监控集群中各个机器的状态，可以发现机器故障，并通知HBase Master。
> + 负责HBase Master的恢复工作（领导选举）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此部分内容同上，在另一篇博文里有详细介绍：[ZooKeeper环境配置](http://blog.csdn.net/wk51920/article/details/51760644)。

# HBase环境配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HBase使用典型的主从架构，集群中只有一个正在运行的HBase Master，剩下均为HRegion Server。因此，完全分布式配置需要在集群中的每台机器中均有相应的配置才行。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 下载安装：前往官网[点此进入官网下载列表](https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/)，选择合适的版本下载。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.  解压缩：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;进入到压缩文件所在的目录，执行如下命令：

``` shell
# 将压缩文件解压缩到当前目录下
tar -zxvf hbase-1.1.5-bin.tar.gz
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 将解压后的文件夹移动到合适的最终目录（也可不移动，不过记录好此时的目录，配置环境时有用）：

```
# 文件夹改名，方便之后配置，也可忽略此步
mv hbase-1.1.5 hbase
# 将文件夹移动到合适的最终位置，可忽略此步
mv -r hbase /usr/hadoop/hadoop-2.6.2/
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. 配置系统环境变量，此步骤分别需要在mac系统和两个ubuntu系统上分别配置：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mac系统上：

```
vim ~/.bash_profile
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;添加：

```
# /usr/hadoop/hadoop-2.6.2/hbase是你之前解压缩后，hbase文件夹根目录的位置
export HBASE_HOME=/usr/hadoop/hadoop-2.6.2/hbase
export PATH=$PATH:$HBASE_HOME/bin:$HBASE_HOME/conf
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;保存退出，执行：`source ~/.bash_profile `使配置立即生效。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ubuntu系统上：

```
sudo vim /etc/profile
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;填入相同的内容，并保存退出，输入`source /etc/profile`使其生效。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5. 配置HBase环境，进入`$HBASE_HOME/conf`目录下，用vim打开`hbase-env.sh`文件，加入如下内容：

```

# 该路径是你本地机器上java目录的路径，在不同系统上此路径内容可能不同
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_73.jdk/Contents/Home

export HBASE_CLASSPATH=/usr/hadoop/hadoop-2.6.2/hbase/conf

export HBASE_OPTS="-XX:+UseConcMarkSweepGC"

export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"

export CLIENT_GC_OPTS="-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/usr/hadoop/hadoop-2.6.2/hbase/logs/gc-hbase.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=1 -XX:GCLogFileSize=512M"

# 此值设为false将不启用hbase自带的zookeeper，而使用我们自己的zookeeper，并需要手动启动zookeeper
export HBASE_MANAGES_ZK=false

```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6. 打开同目录的`hbase-site.xml`文件，修改其中的内容如下：

```
<configuration>
<property>
  <name>hbase.rootdir</name>
  # 注意此值是设置HDFS的访问入口，根据你自己的Hadoop环境而定
  <value>hdfs://10.211.55.2:8020/hbase</value>
 </property>
 
 # 此属性设置为true表示使用完全分布式模式
 <property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
 </property>
 
 # 此值设置了zookeeper集群中的节点，一般与hadoop、HBase使用到的机器相同
 <property>
  <name>hbase.zookeeper.quorum</name>
  <value>master,node1,node2</value>
 </property>
 
 <property>
  <name>hbase.tmp.dir</name>
  <value>file:/usr/hadoop/hadoop-2.6.2/hbase/tmp</value>
 </property>  
 
 <property>
  <name>hbase.master</name>
  <value>hdfs://10.211.55.2:60000</value>
 </property>  
 
 <property>
  <name>hbase.zookeeper.property.dataDir</name>
  <value>file:/usr/hadoop/hadoop-2.6.2/zookeeper</value>
 </property>   
 
 <property>
  <!--htable.setWriteBufferSize(5242880);//5M -->
  <name>hbase.client.write.buffer</name>
  <value>5242880</value>
 </property>   
 
 <property>
  <name>hbase.regionserver.handler.count</name>
  <value>300</value>
  <description>Count of RPC Listener instances spun up on RegionServers.Same property is used by the Master for count of master handlers.</description>
 </property>  
 
 <property>
  <name>hbase.table.sanity.checks</name>
  <value>false</value>
 </property>  
 
 <property>
    <!--every 30s,the master will check regionser is working -->
    <name>zookeeper.session.timeout</name>
    <value>30000</value>
 </property> 
 
 <property>
    <!--every region max file size set to 4G-->
    <name>hbase.hregion.max.filesize</name>
    <value>10737418240</value>
  </property>
</configuration>
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7. 打开同目录下的`regionservers`文件，在其中添加HRegion Server的hostname：（同一台机器上可以同时作为HMaster和HRegion，如果想在master上也运行HRegion Server，则在下面将`master`也添加进去即可）


```
node1
node2
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8. 复制hbase及其配置：


```
# 其中，第一个/usr/hadoop/hadoop-2.6.2/hbase是本机中，hbase的根目录
# wk51920是登录另外两个ubuntu系统的用户名，node1，node2分别是两个ubuntu系统的hostname。
# 之后的/usr/hadoop/hadoop-2.6.2/是node1、node2中将要存放hbase文件夹的位置
scp -r /usr/hadoop/hadoop-2.6.2/hbase wk51920@node1:/usr/hadoop/hadoop-2.6.2/
scp -r /usr/hadoop/hadoop-2.6.2/hbase wk51920@node2:/usr/hadoop/hadoop-2.6.2/
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意：有些关于路径的各种配置，需要在各个系统中根据实际情况进行修改。如果三个系统关于路径的所有位置都相同（如三个系统的java安装目录均为：/Library/Java/JavaVirtualMachines/jdk1.8.0_73.jdk/Contents/Home），则无需再进行修改。


# 运行集群

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;系统启动顺序如下：`hdfs->zookeeper->hbase`。每步启动后，最好先验证一下，保证正常运行，否则出现问题，容易扩大范围，各自的验证方法，在本人相关环境的配置中有说明。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 启动HDFS，进入`$HADOOP_HOME/sbin`目录下:

```
start-all.sh
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 启动ZooKeeper（需要在三个系统中分别手动启动）：

```
zkServer.sh start
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 启动HBase：

```
start-hbase.sh
```

# 验证系统是否正常启动

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;完成上述启动过程后，在控制台输入：


```
hbase shell
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有如下提示：

![这里写图片描述](http://img.blog.csdn.net/20160628171720076)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;输入命令，有如下提示：


```
status
```

![这里写图片描述](http://img.blog.csdn.net/20160628171849750)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;启动成功!

# 注意事项及若干问题

+ ## 进入`hbase shell`后，输入任何命令提示`master is initilizing`错误

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一种解决办法：分别打开三个系统的`/etc/hosts`文件，并将所有关于`127.0.0.1`的行删除，保存退出后再试，我的是这么解决的。

+ ## 进行表的`put`等操作时，提示`xxxx connection refused`或类似的提示，无法完成`put`操作

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;造成这个问题的原因可能是对于域名解析有问题，建议将配置文件中所有的`hostname`（主机名，如master,node1,node2）换成相应的ip地址，再试试，之前在ZooKeeper的配置时，就遇到了类似的问题。