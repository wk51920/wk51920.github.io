---
layout: post
title: 使用Intellij IDEA 16.1 写hadoop程序
---

# 背景

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;前段时间配置好了基于mac的hadoop完全分布式环境，一直想着怎么样去用编译器写程序然后直接在hadoop环境中运行呢，经过一番摸索，写下此文章分享交流。

# 环境介绍

> + os x EI Capitan 10.11.5
> + 虚拟机：Parallels Desktop
> + ubuntukylin 14.04 64bit * 2
> + hadoop 2.6.2
> + os x上的jdk版本：1.8.0_73，ubuntu上的jdk版本：1.8.0_91（不同机器上的jdk版本不要求一样）
> + IntelliJ IDEA 16.1

#### 如果没有配置好hadoop的环境，请参考本人另一篇博文：[http://blog.csdn.net/wk51920/article/details/51686038](http://blog.csdn.net/wk51920/article/details/51686038)

# 下载及安装IntelliJ IDEA 16.1

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 登录官网：[https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)下载最新版本的编译器。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 安装编译器。

# 创建hadoop工程

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 打开IntelliJ IDEA编译器，创建一个新的工程，点击next

![这里写图片描述](http://img.blog.csdn.net/20160615232457648)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 输入工程的名字，点击Finish

![这里写图片描述](http://img.blog.csdn.net/20160615232731524)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 创建如下图的包结构及WordCount.java文件：
![这里写图片描述](http://img.blog.csdn.net/20160615233423402)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. 添加JDK，如果已经配置了JDK可以省略此步骤：在侧边栏右击!

[这里写图片描述](http://img.blog.csdn.net/20160615233517597)选择“Open Module Settings”

![这里写图片描述](http://img.blog.csdn.net/20160615233741840)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;点击“news”选择jdk的根目录即可。

![这里写图片描述](http://img.blog.csdn.net/20160615234006369)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5. 添加hadoop各种jar包

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开“Module Settings”点击左侧边栏中的Libraries。

![这里写图片描述](http://img.blog.csdn.net/20160615234327296)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;点击右侧的“+”号，并选择“Java”，在路径中选择hadoop的各种jar包，我的hadoop jar包路劲为：/usr/hadoop/hadoop-2.6.2/share/hadoop下的所有目录中的jar包。另外还需要特别将/usr/hadoop/hadoop-2.6.2/share/hadoop/common/lib下的所有jar包添加进工程，并点击“Apply”或“OK”。

![这里写图片描述](http://img.blog.csdn.net/20160615234555359)

![这里写图片描述](http://img.blog.csdn.net/20160615234949085)

# 配置编译环境

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 点击右上角的
![这里写图片描述](http://img.blog.csdn.net/20160615235415050)
中的下拉箭头，点击“Edit Configurations...”，并点击左上角的“+”号，并选择“Application”

![这里写图片描述](http://img.blog.csdn.net/20160615235725364)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 在“Main class”中填写我们需要运行的程序：com.wk51920.WordCount;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 在“Program arguments”中填写程序需要的“输入文件”和“输出文件”。在此程序中，“输入文件”即需要用WordCount去统计单词的源文件的位置，“输出文件”即保存统计结果的文件。此栏的完整内容为：hdfs://master:8020/input/README.txt hdfs://master:8020/output/。这两个路径根据你配置的hadoop环境相关，即core-site.xml配置文件中的fs.defaultFS属性的值，即是HDFS系统的根目录。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在“Name”处改为：hadoop,点击“Apply”。

![这里写图片描述](http://img.blog.csdn.net/20160616000927594)

# 填写WordCount代码

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选择WordCount文件，写入如下代码：

```
package com.wk51920;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.*;

import java.io.IOException;
import java.util.Iterator;
import java.util.StringTokenizer;

/**
 * Created by wk51920 on 16/6/3.
 */
public class WordCount  {
    public static class Map extends MapReduceBase implements Mapper<LongWritable, Text, Text, IntWritable>{
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
        @Override
        public void map(LongWritable longWritable, Text text, OutputCollector<Text, IntWritable> outputCollector, Reporter reporter) throws IOException {
            String line = text.toString();
            StringTokenizer tokenizer = new StringTokenizer(line);
            while(tokenizer.hasMoreTokens()){
                word.set(tokenizer.nextToken());
                outputCollector.collect(word,one);
            }
        }
    }

    public static class Reduce extends MapReduceBase implements Reducer<Text,IntWritable,Text, IntWritable>{
        @Override
        public void reduce(Text text, Iterator<IntWritable> iterator, OutputCollector<Text, IntWritable> outputCollector, Reporter reporter) throws IOException {
            int sum= 0;
            while(iterator.hasNext()){
                sum += iterator.next().get();
            }
            outputCollector.collect(text, new IntWritable(sum));
        }
    }

    public static void main(String[] args) throws Exception{
        JobConf conf = new JobConf(WordCount.class);
        conf.setJobName("wordCount");
        conf.setOutputKeyClass(Text.class);
        conf.setOutputValueClass(IntWritable.class);

        conf.setMapperClass(Map.class);
        conf.setReducerClass(Reduce.class);

        conf.setInputFormat(TextInputFormat.class);
        conf.setOutputFormat(TextOutputFormat.class);

        FileInputFormat.setInputPaths(conf,new Path(args[0]));
        FileOutputFormat.setOutputPath(conf, new Path(args[1]));

        JobClient.runJob(conf);
    }

}

```

# 将文件放入HDFS系统

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果没有启动hadoop服务，先启动服务。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 在hdfs系统中建立input文件夹：

```
hadoop fs -mkdir /input  
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 将本地的README.txt放入HDFS中的input文件夹下：

```
hadoop fs -copyFromLocal README.txt /input
```

### 注意运行此程序前，在HDFS系统上不能存在output文件夹，如果存在请删除再运行程序。

# 运行WordCount程序

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 选择WordCount文件，并在代码区右击，选择“Run ‘hadoop’”

![这里写图片描述](http://img.blog.csdn.net/20160616002221754)
