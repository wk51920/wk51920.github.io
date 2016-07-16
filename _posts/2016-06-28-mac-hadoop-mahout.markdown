---
layout: post
title: Mac系统下，Hadoop 2.6.2 + Mahout 0.12.1 完全分布式配置
---

# 简介

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Mahout 是 Apache Software Foundation（ASF） 旗下的一个开源项目，提供一些可扩展的机器学习领域经典算法的实现，旨在帮助开发人员更加方便快捷地创建智能应用程序。Mahout包含许多实现，包括聚类、分类、推荐过滤、频繁子项挖掘。此外，通过使用 Apache Hadoop 库，Mahout 可以有效地扩展到云中。

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上来自“百度百科”

# 搭建环境

> + mac OSX EI Capitan 10.11.5 (master节点)
> + 虚拟机：Parallels Desktop
> + ubuntukylin 14.04 64bit * 2 (node1节点，node2节点)
> + Hadoop 2.6.2
> + Mahout 0.12.1

# 配置Hadoop环境

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;请参考博文：[Hadoop完全分布式配置](http://blog.csdn.net/wk51920/article/details/51686038)。

# 安装并配置Mahout环境

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 下载安装包：[点此选择想要下载的版本](http://www-eu.apache.org/dist/mahout/)。有两种类型，一种是编译好的项目，另一种是源代码，需要自己在本地编译。我们采用的是已经编译好的版本（就是压缩包名没有“src”的那个）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 在合适的目录解压缩，并改名：

```
tar -zxvf apache-mahout-distribution-0.12.1.tar.gz
mv apache-mahout-distribution-0.12.1 mahout
# 将解压后并改名后的文件夹移动至/usr/hadoop/hadoop-2.6.2/目录下（个人习惯，可忽略）
mv -r mahout /usr/hadoopp/hadoop-2.6.2/
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 配置环境变量：

```
vim ~/.bash_profile
```

```
# 若已存在则只添加没有的条目，特别是HADOOP_CONF_DIR一般之前没有配置
export HADOOP_HOME="/usr/hadoop/hadoop-2.6.2"
export HADOOP_CONF_DIR=/usr/hadoop/hadoop-2.6.2/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/etc/hadoop
export CLASSPATH=$CLASSPATH:$($HADOOP_HOME/bin/hadoop classpath) 

export MAHOUT_HOME=/usr/hadoop/hadoop-2.6.2/mahout
export MAHOUT_CONF_DIR=$MAHOUT_HOME/conf
export PATH=$PATH:$MAHOUT_HOME/conf:$MAHOUT_HOME/bin
```

```
source ~/.bash_profile
```

# 运行测试程序

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 启动Hadoop：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;进入$HADOOP_HOME/sin


```
start-all.sh
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 在HDFS上创建程序需要的输入文件的位置：


```
# 此目录根据运行程序的不同而提示不同
# wk51920是我登陆系统的用户名
hadoop fs -mkdir /user
hadoop fs -mkdir /user/wk51920
hadoop fs -mkdir /user/wk51920/testdata
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 下载测试数据：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开网址：[http://archive.ics.uci.edu/ml/databases/synthetic_control/](http://archive.ics.uci.edu/ml/databases/synthetic_control/)，选择`synthetic_control.data`下载。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. 上传测试数据至HDFS：


```
hadoop fs -put synthetic_control.data的位置 /user/wk51920/testdata/
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5. 运行测试程序：


```
mahout -core  org.apache.mahout.clustering.syntheticcontrol.kmeans.Job
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;等几分钟，会将结果输出至控制台。

# 注意事项

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;很多测试程序由于Mahout版本的升级而废弃了，所以如果按照网上老版本的教程无法运行一些程序或命令，有可能是这个原因。