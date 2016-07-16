---
layout: post
title: Mac系统下, Hdoop 2.6.2 + Hive 2.0.1 配置
---

# 简介(百度百科)

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。 其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。
>  


# 环境介绍

> + 操作系统：OSX EI Capitan 10.11.5
> + Hadoop版本：2.6.2
> + mySql版本：5.6.21
> + mysql-connector-java版本：5.1.38
> + Hive版本：2.0.1

# Hadoop配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;关于hadoop的详细配置，请移步博文：[Hadoop2.6.2完全分布式配置](http://blog.csdn.net/wk51920/article/details/51686038)。

# 下载安装

### 安装mySql

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 先去mySql官网[http://dev.mysql.com/downloads/mysql/](http://dev.mysql.com/downloads/mysql/)下载安装包。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 双击安装包：mysql-5.1.38-osx10.11-x86_64.dmg（具体看你下载的版本）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 按照提示界面一路安装即可，此处注意：在安装结束时，可能会弹出对话框，告知你默认的访问数据库的用户名和密码。第一次登陆时，需要用此用户名和密码登陆，否则会出现无法登陆的问题。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. 修改用户密码：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. 用刚才的用户名和密码登陆mysql：


```
mysql -u 用户名 -p 密码
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. 输入下列语句：

```
#第一条命令，修改密码
UPDATE user SET password=PASSWORD('新的密码') WHERE user='你登录的用户名';
#第二条命令，使改动立即生效
FLUSH PRIVILEGES;
```

### 安装Hive

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 先确保已经正确安装并运行了hadoop。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 下载Hive安装包

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;去官网[https://hive.apache.org/downloads.html](https://hive.apache.org/downloads.html)下载合适的安装包版本，将安装包移动至：`/usr/hadoop/hadoop-2.6.2/` 目录下，此目录是本机安装hadoop的目录。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;移动至此处后，解压缩，并将解压后的文件名改为hive，方便配置。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;例如本机Hive的安装路径为：`/usr/hadoop/hadoop-2.6.2/hive` 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 配置系统环境变量

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. 修改~/.bash_profile文件或者修改/etc/profile文件

```
vim ~/.bash_profile
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. 修改内容为：

``` properties
export HIVE_HOME=/usr/hadoop/hadoop-2.6.2/hive
export PATH=$PATH:$HIVE_HOME/bin:$HIVE_HOME/conf
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c. 退出保存后，在终端输入，使环境变量立即生效：

```
source ~/.bash_profile
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. 修改Hive配置文档：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. 进入`/usr/hadoop/hadoop-2.6.2/hive/conf`，新建文件`hive-site.xml`     

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. 添加`hive-site.xml`内容：

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>hive.metastore.local</name>
        <value>true</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://master:3306/hive?createDatabaseIfNotExist=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;autoReconnect=true&amp;useSSL=false</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>root</value>
    </property>
</configuration>
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c. 复制`hive-env.sh.template`为`hive-env.sh`


```
cp hive-env.sh.template hive-env.sh
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;d. 修改`hive-env.sh`内容：

```
HADOOP_HOME=/usr/hadoop/hadoop-2.6.2
export HIVE_CONF_DIR=/usr/hadoop/hadoop-2.6.2/hive/conf
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5. mySql配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. 登陆mySql，我的用户名为root，密码为root。

```
mysql -u root -p root
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. 给用户赋予权限，以使得该用户可以远程登录数据库：

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c. 使改变立即生效：

```
FLUSH PRIVILEGES;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6. 向`/usr/hadoop/hadoop-2.6.2/hive/lib`中添加mySql连接库：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. 去网站[http://dev.mysql.com/downloads/connector/j/](http://dev.mysql.com/downloads/connector/j/)下载mySql-connector包。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. 将下好的包解压缩，如我解压缩后的文件夹为`mysql-connector-java-5.1.38`，将此文件夹下`mysql-connector-java-5.1.38-bin.jar`复制到`/usr/hadoop/hadoop-2.6.2/hive/lib`下。

**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意：需要给`/tmp`文件夹设置写权限，同时确保 hadoop不在安全模式下，可以执行此命令使hadoop退出安全模式：`hadoop dfsadmin -safemode leave`**

# 启动Hive

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 如果是第一次启动Hive，则需要先执行如下初始化命令：


```
schematool -dbType mysql -initSchema
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 启动Hive:


```
hive
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;完成基本的环境配置！
