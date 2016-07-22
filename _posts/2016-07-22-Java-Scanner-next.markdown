---
layout: post
title: Scanner中next*(), next(), nextLine()混用的问题
---
# 背景

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本人最近在做华为机试练习题时，由于其输入用例为多行数据，因此混用了Scanner的next*()、next()、nextLine()。在使用过程中，发现了混用的一些问题，特此记录并分享。

# 三者的区别

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Scanner内部拥有一个缓冲区，并且存在一个指针p指向下一个将要读取的元素，每调用一次next()、next*()、nextLine()，指针p都将会向前移动特定的距离。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. next()划分每个元素的标准是：空格、制表符、或者换行符。所有元素均有这三种情况分割开来，其所有返回的值均为String类型。对于如下输入：

```
10 23 42 44
12 20
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;调用第一次next()，将会返回String类型的“10”，第二次调用返回“23”，依次类推，不同行的内容会自动跳至下一行的开始元素，如：第四次调用next()时返回“44”，而第五次调用next()，会返回“12”，并不会因为换行而造成返回换行符的情况。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;测试程序如下：


```
import java.util.Scanner;

public class Test {
    public static void main(String[] args) throws Exception {
        Scanner in = new Scanner(System.in);
        while (in.hasNext()) {
            for(int i = 0; i<6;i++)
                System.out.println("第"+(i+1)+"个输入,值为: "+in.next());
        }
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;控制台输入输出显示内容如下：


```
10 23 42 44
12 20
第1个输入,值为: 10
第2个输入,值为: 23
第3个输入,值为: 42
第4个输入,值为: 44
第5个输入,值为: 12
第6个输入,值为: 20
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. next*()与next()的区别在于，可以将下一个需要读取的元素直接转化为指定的类型（如果可以转化的话），其余特性均相同。对于如下数据：


```
10 23 hello
world 2.34
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第一次调用nextDouble()会自动将整型10转化为值为10.0的double类型，第二次调用nextDouble()会自动将整型23转化为值为23.0的double类型，第三次调用nextDouble()会产生`java.util.InputMismatchException`异常，因为无法将一个字符串转化为一个double类型。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;测试程序如下：


```
import java.util.Scanner;

public class Test {
    public static void main(String[] args) throws Exception {
        Scanner in = new Scanner(System.in);
        while (in.hasNext()) {
            System.out.println(in.nextDouble()+"");
        }
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;控制台输入输出内容如下：


```
10 23 hello
world 2.34
10.0
23.0
Exception in thread "main" java.util.InputMismatchException
	at java.util.Scanner.throwFor(Scanner.java:864)
	at java.util.Scanner.next(Scanner.java:1485)
	at java.util.Scanner.nextDouble(Scanner.java:2413)
	at Test.main(Test.java:9)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. nextLine()会按照换行符来对所有输入进行分割，每调用一次nextLine()，会返回下一行的内容，返回类型为String。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于如下输入：


```
100 23 20 hello world 245
good bye 123.23 24.5 988
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第一次调用nextLine()将返回字符串"100 23 20 hello world 245"，第二次调用nextLine()将返回字符串"good bye 123.23 24.5 988"，且行内的空格如果有多个，也会原样保留。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;测试程序如下：


```
import java.util.Scanner;

public class Test {
    public static void main(String[] args) throws Exception {
        Scanner in = new Scanner(System.in);
        while (in.hasNext()) {
            System.out.println(in.nextLine());
        }
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;控制台输入输出内容如下：


```

# 第一组输入输出
100 23 20 hello world 245
good bye 123.23 24.5 988
100 23 20 hello world 245
good bye 123.23 24.5 988

# 第二组输入输出，可以看到，行内空格很好地保留了下来
100 23 20 hello world 245
good bye      123.23 24.5 988
100 23 20 hello world 245
good bye      123.23 24.5 988
```

# 三者混用时出现的问题

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们对如下输入样例进行测试：


```
10 20 30 40 hello
world bye
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;测试程序如下；


```
import java.util.Scanner;

public class Test {
    public static void main(String[] args) throws Exception {
        Scanner in = new Scanner(System.in);
        while (in.hasNext()) {
            System.out.println(in.next());
            System.out.println(in.next());
            System.out.println(in.next());
            System.out.println(in.next());
            System.out.println(in.next());
            System.out.println("---------不同调用的分割线--------");
            String line = in.nextLine();
            System.out.println(line);
        }
    }
}

```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;控制台输入输出内容如下：


```
10 20 30 40 hello
world bye
10
20
30
40
hello
---------不同调用的分割线--------

world
bye
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们发现，之前调用next()得到的内容都跟我们预期的相同，但当在调用过next()后再紧接着调用nextLine()就出现了问题，我们的预期是得到字符串"world bye"，而实际得到的字符串是"\nworld\nbye"，不仅在下一行的开头多读入了上一行末尾的换行符，还将本行的各个元素间自动加入了换行符。这个在使用的过程中如果没有预想到，将会在后续的手动分割元素的过程中出现很多问题。比如，按预期，将此行元素分割成字符串数组应该使用`String[] lineStr = line.split(" ")` 即：将此行内容按空格分割，而由我们实际得到的字符串line内容来看，应该按照换行符分割才能得到想要的内容。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果全部使用nextLine()来处理输入，则全部按照预期进行。测试程序如下：


```
import java.util.Scanner;

public class Test {
    public static void main(String[] args) throws Exception {
        Scanner in = new Scanner(System.in);
        while (in.hasNext()) {
            System.out.println(in.nextLine());
            System.out.println(in.nextLine());
        }
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;控制台输入输出内容如下：


```
10 20 30 40 hello
world bye
10 20 30 40 hello
world bye
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;next*()由于跟next()基本相同，所以在与nextLine()混合使用时，也会产生相同的问题，在这里不再举例展示。

# 总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;总结下来就是，使用Scanner处理输入流的过程中，最好不要混合使用，要么全部用nextLine()读取，要么全部用next()/next*()处理，以防出现不可预知的错误。