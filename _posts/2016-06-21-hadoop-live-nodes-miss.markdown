---
layout: post
title: Hadoop datanode正常启动，但是Live nodes中却缺少节点的问题
---

# 背景

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最近在管理集群时发现明明所有数据节点都已经正常启动了，而通过命令`hadoop dfsadmin -report` 显示的 Live datanodes却只有一个。同时，通过web页面查看`http://master:50070`，在Live Node那一栏也显示只有一个节点，点击进入该节点查看情况，发现是node1，但诡异的是：这时候点击刷新，刷新后的live node数仍然为1，但却变为了node2，反复刷新会发现不停地在node1和node2之间切换（由于我只有两个datanode，若超过两个说不定会在所有的datanode间切换）。通过一番摸索，发现了解决方法。

# 环境

> + master一个作为NameNode
> + node1作为DataNode
> + node2作为另一个DataNode


# 解决方法

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 打开配置文件hdfs-site.xml找到`dfs.datanode.name.dir`这个属性，或者`dfs.data.dir`具体看你用哪个设置的数据存储路径。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 分别在master, node1, node2中更改此属性

```
 #master中的值
 <property>
 <name>dfs.datanode.data.dir</name>
 <value>file:/usr/hadoop/hadoop-2.6.2/dfs/data</value>
 </property>
```

```
 #node1中的值
 <property>
 <name>dfs.datanode.data.dir</name>
 <value>file:/usr/hadoop/hadoop-2.6.2/dfs/data/node1</value>
 </property>
```

```
 #node2中的值
 <property>
 <name>dfs.datanode.data.dir</name>
 <value>file:/usr/hadoop/hadoop-2.6.2/dfs/data/node2</value>
 </property>
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其实关键就是多个节点存放data数据的目录路径相同了，造成了报告中误认为只有一个datanode！