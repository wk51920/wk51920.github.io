---
layout: post
title: Mac系统下，Hadoop 2.6.2 + Zookeeper 3.4.6 完全分布式配置
---

# 简介

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ZooKeeper是一个为分布式应用所设计的开源协调服务。它可以为用户提供同步、配置管理、分组和命名等服务。用户可以使用ZooKeeper提供的接口方便地实现一致性、组管理、leader选举及某些协议。ZooKeeper意欲提供一个易于编程的环境，所以它的文件系统使用了我们所熟悉的目录树结构。ZooKeeper是使用Java编写的，但是它支持Java和C两种编程语言接口。
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;众所周知，协调服务非常容易出错，而且很难从故障中恢复，例如，协调服务很容易处于竞态以至于出现死锁。ZooKeeper的设计目的是为了减轻分布式应用程序所承担的协调任务。
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上介绍来自《Hadoop实战》

# 环境

> + mac OSX EI Capitan 10.11.5  (master节点)
> + 虚拟机：Parallels Desktop
>+ ubuntukylin 14.04 64bit * 2     (node1节点，node2节点)
>+ ZooKeeper 3.4.6

# Hadoop 安装

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;关于此部分内容，请移步博文：[Hadoop 2.6.2 完全分布式配置](http://blog.csdn.net/wk51920/article/details/51686038)。

# ZooKeeper安装

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该配置过程需要在每个系统上均实现一遍，也可以在mac上先配置好，然后通过`scp`命令复制到其余的节点上，再做些许修改即可。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下述步骤需要保证三个节点间能互相通信，并且可以通过SSH免密码登陆，如有疑问可参考本人另一篇博文：[http://blog.csdn.net/wk51920/article/details/51686038](http://blog.csdn.net/wk51920/article/details/51686038)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 前往ZooKeeper官网，下载3.4.6安装包：[http://apache.fayea.com/zookeeper/](http://apache.fayea.com/zookeeper/)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 默认将安装包下载至~/Downloads下，可以在此目录下解压缩：


```
tar -zxvf zookeeper-3.4.6.tar.gz 
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;得到文件夹`zookeeper-3.4.6`。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 可以将此文件夹移动至合适的目录下，也可以不移动，记住路径即可，如在mac系统上，我的存放路径是：`/usr/hadoop/hadoop-2.6.2/`，并将`zookeeper-3.4.6`文件夹更名为`zookeeper`方便管理。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. 配置环境变量：


```
vim ~/.bash_profile
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在其中加入：


```
#其中的/usr/hadoop/hadoop-2.6.2/zookeeper是我存放zookeeper的路径
#注意其中的zookeeper是zookeeper-3.4.6解压并改名后的根目录
export ZOOKEEPER_HOME=/usr/hadoop/hadoop-2.6.2/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;保存后退出，并使用source命令使配置立即生效：


```
source ~/.bash_profile
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5. 进入$ZOOKEEPER_HOME/conf文件夹下，其中有文件`zoo_sample.cfg`文件，执行如下命令：


```
cp zoo_sample.cfg zoo.cfg
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;复制一份配置文件，打开`zoo.cfg`，在其中完成如下修改：


```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/usr/hadoop/hadoop-2.6.2/zookeeper/data
dataLogDir=/usr/hadoop/hadoop-2.6.2/zookeeper/logs
# the port at which the clients will connect
clientPort=2181

server.1=10.211.55.2:2888:3888
server.2=10.211.55.6:2888:3888
server.3=10.211.55.7:2888:3888
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
autopurge.purgeInterval=1

```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其中：


```
# 这三个是集群中的三个节点的ip地址及端口号，2888用于连接leader的端口，用于服务器间互连
# 3888是进行leader选举时用到的端口
# 注意数字1，2，3分别要与$ZOOKEEPER_HOME/data/myid中的数字相对应（其中data目录和myid文件需要自己建立，比如10.211.55.2对应的mac系统的ip地址，由于它是server.1的值，所以在mac系统下，$ZOOKEEPER_HOME/data/myid内的值为1，以此类推）
server.1=10.211.55.2:2888:3888
server.2=10.211.55.6:2888:3888
server.3=10.211.55.7:2888:3888
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6. 进入到$ZOOKEEPER_HOME目录下，创建两个文件夹：


```
mkdir data
mkdir logs
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;进入data目录下，创建`myid`文件，并在里面写入`1`，保存退出。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7. 进入到`/usr/hadoop/hadoop-2.6.2`目录下，该目录下有文件夹`zookeeper`，执行命令：


```
# 将在Mac上配置好的zookeeper复制到node1和node2上。
# wk51920是登陆操作系统的用户名，node1和node2分别是两个ubuntu系统的hostname
# 后面的路径是存放在各自节点下的路径
scp -r zookeeper wk51920@node1:/usr/hadoop/hadoop-2.6.2/
scp -r zookeeper wk51920@node2:/usr/hadoop/hadoop-2.6.2/
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8. 打开node1、node2系统，修改各自的`/etc/profile`文件，配置环境变量，内容同mac。在各自系统的$ZOOKEEPER_HOME下创建`data`和`logs`文件夹，并在node1的data文件夹下创建文件`myid`，在内写入`2`。同样在node2的data文件夹下建立文件`myid`，并在其中写入`3`，主要是与`server.x`相对应。

# 启动ZooKeeper

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在各自系统的控制台输入：


```
zkServer.sh start
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果想要在控制台看到启动信息，可以输入：


```
zkServer.sh start-foreground
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意：刚开始可能有异常信息，那是因为只启动了部分节点，无法达到集群对节点数的要求，当所有节点均启动后，异常会恢复。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在终端输入：


```
zkServer.sh status
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果显示如下类似信息，则证明已正常启动：

![这里写图片描述](http://img.blog.csdn.net/20160625225642088)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;或者：

![这里写图片描述](http://img.blog.csdn.net/20160625225757418)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表明启动成功！

# 注意事项

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果在执行


```
zkServer.sh start
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;提示如下信息：

![这里写图片描述](http://img.blog.csdn.net/20160625225945201)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是执行


```
zkServer.sh status
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;时，却提示：

![这里写图片描述](http://img.blog.csdn.net/20160625230149552)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;并且查看日志，或者在控制台显示的日志信息一直提示异常：`xxxxx : connection refused`，主要提示无法连接3888或2888时，请确保所有系统的防火墙已关闭，而且，所有系统中配置文件`zoo.cfg`的`server.x=xxxxx:2888:3888`使用的是具体的ip值，而不是hostname。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之前我的配置使用的是hostname，就一直报这个错误，原来出问题的配置书写方式如下：


```
server.1=master:2888:3888
server.2=node1:2888:3888
server.3=node2:2888:3888
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;只要将其中的各个hostname改成实际的ip值即可。