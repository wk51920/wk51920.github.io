---
layout: post
title: Java中创建单例模式的五种方法及线程安全
---

# 背景
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;单例模式在日常工程项目中应用十分广泛，但是如果没有考虑线程安全的问题，可能会在多线程环境下产生错误的结果，生成多个实例。

# 公共类
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以下两个类，是本次实验中用到的两个公共类，SingletonFactory用于创建各种单例对象，SingletonTest用于测试各种单例模式的线程安全性。
```
public interface SingletonFactory {
    public static SingletonImit getSingletonImit() {
        return SingletonImit.getInstance();
    }

    public static SingletonDCL getSingletonDCL() {
        return SingletonDCL.getInstance();
    }

    public static SingletonStaticBlock getSingletonStaticBlock() {
        return SingletonStaticBlock.getInstance();
    }

    public static SingletonInnerStaticClass getSingletonInnerStaticClass() {
        return SingletonInnerStaticClass.getInstance();
    }

    public static SingletonInnerEnum getSingletonInnerEnum() {
        return SingletonInnerEnum.getInstance();
    }

    public static SingletonWrong getSingletonWrong() {
        return SingletonWrong.getInstance();
    }
}
```

```
public class SingletonTest {
    public static void main(String[] args) {
        Thread[] threads = new Thread[5];
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("线程:" + Thread.currentThread().getName() + "创建了单例对象, 其hashcode值为:" + SingletonFactory.getSingletonWrong().hashCode());
            }
        };

        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(runnable);
            threads[i].setName(i + 1 + "");
            threads[i].start();
        }

    }
}
```

# 一个错误的示范
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以下将要展示的是一种错误的单例模式。

```
public class SingletonWrong {

    private volatile static SingletonWrong instance;

    private SingletonWrong() {

    }

    public static SingletonWrong getInstance() {
        try {
            if (instance == null) {
                Thread.sleep(3000);
                instance = new SingletonWrong();

            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return instance;
    }

}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该程序在测试环境中运行结果如下：

![这里写图片描述](http://img.blog.csdn.net/20160904172102436)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从hashcode我们可以看出，多线程使用单例模式时，此种方法并没有达到单例的效果，而是创建了多个实例。

# 五种线程安全的单例模式实现方式

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 使用“立即加载”模式：

```
public class SingletonImit {
    private static SingletonImit instance = new SingletonImit();

    private SingletonImit() {
    }

    public static SingletonImit getInstance() {
        return instance;
    }

}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该实现在测试环境中运行结果如下：

![这里写图片描述](http://img.blog.csdn.net/20160904172456837)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从结果上我们可以看到，多线程下达到了获得单例的目的。不过此种方式的缺陷在于：在类加载过程中就已经将具体的对象创建完毕，如果对象很大，会提前一直占用着内存。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 使用“延时加载”实现线程安全的单例模式：

```
public class SingletonDCL {
    private volatile static SingletonDCL instance;

    private SingletonDCL() {

    }

    public static SingletonDCL getInstance() {
        try {
            if (instance == null) {
                Thread.sleep(3000);
                synchronized (SingletonDCL.class) {
                    if (instance == null)
                        instance = new SingletonDCL();
                }
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return instance;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在测试环境中运行结果如下：
![这里写图片描述](http://img.blog.csdn.net/20160904172901955)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此种方式实际上是对上面展示的“错误示范”的一种改进，可以看到，新的实现方式达到了单例的目的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 利用静态内部类实现线程安全的单例模式：

```
public class SingletonInnerStaticClass implements Serializable {
    private static final long serialVersionUID = 888L;

    private SingletonInnerStaticClass() {
    }

    private static class SingleHandler {
        private static SingletonInnerStaticClass instatnce = new SingletonInnerStaticClass();

    }

    public static SingletonInnerStaticClass getInstance() {
        return SingleHandler.instatnce;
    }

    protected Object readResolve() throws ObjectStreamException {
        return SingleHandler.instatnce;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该实现方式在测试环境中运行结果如下：

![这里写图片描述](http://img.blog.csdn.net/20160904173444566)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此种方式需要注意的是在单例对象需要序列化时，需要重写`protect Object readResolve()`方法。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. 利用“静态代码块”实现单例模式：

```
public class SingletonStaticBlock {
    private static SingletonStaticBlock instance;

    static {
        instance = new SingletonStaticBlock();
    }

    private SingletonStaticBlock() {
    }

    public static SingletonStaticBlock getInstance() {
        return instance;
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 此实现方式在测试环境中运行结果如下：

![这里写图片描述](http://img.blog.csdn.net/20160904174015287)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此种实现方法与“立即加载”的原理基本相同，都是利用类加载时的初始化操作实现的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5. 利用“内部枚举类型”实现线程安全的单例模式：

```
public class SingletonInnerEnum {
    private SingletonInnerEnum() {
    }

    public enum Instance {
        SingleinnerEnum;
        private SingletonInnerEnum instance;

        private Instance() {
            try {
                Thread.sleep(3000);
                instance = new SingletonInnerEnum();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        public SingletonInnerEnum getInstance() {
            return instance;
        }
    }

    public static SingletonInnerEnum getInstance() {
        return Instance.SingleinnerEnum.getInstance();
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此方法在测试环境中结果如下:

![这里写图片描述](http://img.blog.csdn.net/20160904174332800)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以从结果看到，此种方式确实能够在多线程环境下实现单例的目的。

# 结语

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面虽然介绍了五种实现单例模式的方法，但只是具体的实现方式不同，实质上只有“立即加载”与“延迟加载”两种。特别需要注意的是使用“延迟加载”时，需要两重检测，否则即使存在同步块，也无法保证线程安全。
