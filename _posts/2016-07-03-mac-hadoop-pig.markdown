---
layout: post
title: Mac系统下，Hadoop 2.6.2 + Pig 0.16.0 安装配置
---

# 简介

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;作为Apache项目的一个子项目，Pig提供了一个支持大规模数据分析的平台。Pig包括用来描述数据分析程序的高级程序语言，以及对这些程序进行评估的基础结构。Pig突出的特点就是它的结构经得起大量并行任务的检验，这使得它能够处理大规模数据集。

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上来自《Hadoop实战》

# 环境

> * mac OSX EI Capitan 10.11.5 (master节点)
> * 虚拟机：Parallels Desktop
> * ubuntukylin 14.04 64bit * 2 (node1节点，node2节点)
> * Hadoop 2.6.2
> * Pig 0.16.0

# Hadoop配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;请参考博文：[Hadoop配置](http://blog.csdn.net/wk51920/article/details/51686038)。

# Pig配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 下载安装Pig：[Pig0.16.0下载页面](http://mirrors.hust.edu.cn/apache/pig/)。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 将压缩包在合适的位置解压缩，如本机在mac系统上:`/usr/hadoop/hadoop-2.6.2/`下解压：


```
tar -zxvf 压缩包名
# 文件夹改名
mv 压缩包名 pig
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 打开`~/.bash_profile`文件，并添加内容：


```
vim ~/.bash_profile
```

```
# 刚刚pig解压缩并改名后的路径
export PIG_HOME=/usr/hadoop/hadoop-2.6.2/pig
export PATH=$PATH:$PIG_HOME/bin:$PIG_HOME/conf
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. 使配置信息立即生效：

```
source ~/.bash_profile
```

# 运行Pig

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Pig的运行模式有两种：

> * Local模式：它将只访问本地一台主机。
> * MapReduce模式：它将访问一个Hadoop集群和HDFS安装目录。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每种模式下都有三种具体的运行方式：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. Grunt Shell方式：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此方式实际上是通过控制台，逐条输入各种命令，达到对数据的操作过程。


```
# 使用local模式进入Grunt shell
pig -x local

# 使用mapreduce模式进入Grunt shell
pig -x mapreduce
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 脚本文件方式：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此方式是在`*.pig`脚本文件中，一次性将需要执行的命令按顺序写入其中，逐条运行。


```
# script.pig是自己创建的脚本文件
pig -x local script.pig

pig -x mapreduce script.pig
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 嵌入式程序方式：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此方式是通过使用API，在JAVA程序中操作pig数据。

# 测试

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以MapReduce模式测试Pig是否正常安装，需要先正常启动Hadoop。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 创建文件`Student`，并在其中加入如下记录（该数据来自于《Hadoop实战》）：


```
201000101:李勇:男:20:计算机软件与理论
201000102:王丽:女:19:计算机软件与理论
201000103:刘花:女:18:计算机应用技术
201000104:李肖:男:19:计算机系统结构
201000105:吴达:男:19:计算机系统结构
201000106:滑可:男:19:计算机系统结构
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其对应的数据类型如下：


```
Student(Sno:chararray,Sname:chararray,Ssex:chararray,Sage:int,Sdept:chararray)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 使用Grunt Shell执行命令：


```
pig -x mapreduce
```

```
A=load 'Student文件的目录' using PigStorage(':') as (Sno:chararray,Sname:chararray,Ssex:chararray,Sage:int,Sdept:chararray);

B=foreach A generate Sname,Sage;

dump B;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果在控制台正常输出学生表中姓名和年龄两列信息，则成功！