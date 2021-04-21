# Java

## Java基础

------



### 面向对象的三大特性

#### 封装

如与面向过程相比，或者说我们在只看当前代码而不ctrl加左键看源码的情况下，我们是看不到此类的属性和该方法的具体实现细节的。因此封装的作用就是隐藏了对象的属性和实现细节。

在我们看不到对象的属性和方法细节的前提下，那么就有了**安全性**。**简化编程**体现在，与面向过程相比，一个对象调用一个方法只需一个单词，而面向过程则需要将该方法的所有代码复制过来。

#### 继承

不同类型的对象，相互之间经常有一定数量的共同点。通过使用继承，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

#### 多态

具体表现为父类的引用指向子类的实例。

- 如果子类重写了父类的方法，真正执行的是子类覆盖的方法，如果子类没有覆盖父类的方法，执行的是父类的方法。

- 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定。

好处是对于以子类为参数的方法，不需要针对每种子类都写一个方法，而是写一个将父类作为参数的方法即可。

### object类

#### 常见方法

```java

public final native Class<?> getClass()//native方法，用于返回当前运行时对象的Class对象，使用了final关键字修饰，故不允许子类重写。

public native int hashCode() //native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap。
public boolean equals(Object obj)//用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用户比较字符串的值是否相等。

protected native Object clone() throws CloneNotSupportedException//naitive方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为true，x.clone().getClass() == x.getClass() 为true。Object本身没有实现Cloneable接口，所以不重写clone方法并且进行调用的话会发生CloneNotSupportedException异常。

public String toString()//返回类的名字@实例的哈希码的16进制的字符串。建议Object所有的子类都重写这个方法。

public final native void notify()//native方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。

public final native void notifyAll()//native方法，并且不能重写。跟notify一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。

public final native void wait(long timeout) throws InterruptedException//native方法，并且不能重写。暂停线程的执行。注意：sleep方法没有释放锁，而wait方法释放了锁 。timeout是等待时间。

public final void wait(long timeout, int nanos) throws InterruptedException//多了nanos参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上nanos毫秒。

public final void wait() throws InterruptedException//跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念

protected void finalize() throws Throwable { }//实例被垃圾回收器回收的时候触发的操作

```

#### object对象大小

new一个无参的object对象大小为16字节。对象：对象头，实例数据，填充字节。

有指针压缩，Markword占8字节，class point占4字节，无实例数据，对其填充4字节。

无指针压缩，Markword占8字节，class point占8字节，无实例数据，不用对其填充。



### 进程，线程，协程

**进程**：是程序的一次执行过程，是系统运行程序的基本单位。系统运行一个程序即是一个进程从创建，运行到消亡的过程。每个进程还占有某些系统资源如 CPU 时间，内存空间，文件，输入输出设备的使用权等等。

**线程**：是一个比进程更小的执行单位。一个进程可以产生多个线程。同类的多个线程共享该进程的内存空间和系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，线程也被称为轻量级进程。

[线程基本状态](https://snailclimb.gitee.io/javaguide/#/docs/java/basis/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86?id=_332-%e7%ba%bf%e7%a8%8b%e6%9c%89%e5%93%aa%e4%ba%9b%e5%9f%ba%e6%9c%ac%e7%8a%b6%e6%80%81)

**协程**：协程是轻量级的线程，不需要从用户态切换到内核态。

- 线程的切换由操作系统负责调度，协程由用户自己进行调度，减少了上下文切换，提高了效率
- 线程的默认 Stack 是1M，协程更加轻量，是 1K，在相同内存中可以开启更多的协程。
- 由于在同一个线程上，因此可以`避免竞争关系`而使用锁。
- 适用于`被阻塞的`，且需要大量并发的场景。但不适用于大量计算的多线程，遇到此种情况，更好用线程去解决。



### BIO、NIO、AIO

[JAVA中IO流](https://snailclimb.gitee.io/javaguide/#/docs/java/basis/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86?id=_341-java-%e4%b8%ad-io-%e6%b5%81%e5%88%86%e4%b8%ba%e5%87%a0%e7%a7%8d)

NIO使用：

> NIOClient

```java
package com.atguigu.nio;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

public class NIOClient {
    public static void main(String[] args) throws Exception{

        //得到一个网络通道
        SocketChannel socketChannel = SocketChannel.open();
        //设置非阻塞
        socketChannel.configureBlocking(false);
        //提供服务器端的ip 和 端口
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 6666);
        //连接服务器
        if (!socketChannel.connect(inetSocketAddress)) {

            while (!socketChannel.finishConnect()) {
                System.out.println("因为连接需要时间，客户端不会阻塞，可以做其它工作..");
            }
        }

        //...如果连接成功，就发送数据
        String str = "hello, 尚硅谷~";
        //Wraps a byte array into a buffer
        ByteBuffer buffer = ByteBuffer.wrap(str.getBytes());
        //发送数据，将 buffer 数据写入 channel
        socketChannel.write(buffer);
        System.in.read();

    }
}
```

NIOServer：

```java
package com.atguigu.nio;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class NIOServer {
    public static void main(String[] args) throws Exception{

        //创建ServerSocketChannel -> ServerSocket
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        //得到一个Selecor对象
        Selector selector = Selector.open();

        //绑定一个端口6666, 在服务器端监听
        serverSocketChannel.socket().bind(new InetSocketAddress(6666));
        //设置为非阻塞
        serverSocketChannel.configureBlocking(false);

        //把 serverSocketChannel 注册到  selector 关心 事件为 OP_ACCEPT
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("注册后的selectionkey 数量=" + selector.keys().size()); // 1



        //循环等待客户端连接
        while (true) {

            //这里我们等待1秒，如果没有事件发生, 返回
            if(selector.select(1000) == 0) { //没有事件发生
                System.out.println("服务器等待了1秒，无连接");
                continue;
            }

            //如果返回的>0, 就获取到相关的 selectionKey集合
            //1.如果返回的>0， 表示已经获取到关注的事件
            //2. selector.selectedKeys() 返回关注事件的集合
            //   通过 selectionKeys 反向获取通道
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            System.out.println("selectionKeys 数量 = " + selectionKeys.size());

            //遍历 Set<SelectionKey>, 使用迭代器遍历
            Iterator<SelectionKey> keyIterator = selectionKeys.iterator();

            while (keyIterator.hasNext()) {
                //获取到SelectionKey
                SelectionKey key = keyIterator.next();
                //根据key 对应的通道发生的事件做相应处理
                if(key.isAcceptable()) { //如果是 OP_ACCEPT, 有新的客户端连接
                    //该该客户端生成一个 SocketChannel
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    System.out.println("客户端连接成功 生成了一个 socketChannel " + socketChannel.hashCode());
                    //将  SocketChannel 设置为非阻塞
                    socketChannel.configureBlocking(false);
                    //将socketChannel 注册到selector, 关注事件为 OP_READ， 同时给socketChannel
                    //关联一个Buffer
                    socketChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));

                    System.out.println("客户端连接后 ，注册的selectionkey 数量=" + selector.keys().size()); //2,3,4..


                }
                if(key.isReadable()) {  //发生 OP_READ

                    //通过key 反向获取到对应channel
                    SocketChannel channel = (SocketChannel)key.channel();

                    //获取到该channel关联的buffer
                    ByteBuffer buffer = (ByteBuffer)key.attachment();
                    channel.read(buffer);
                    System.out.println("form 客户端 " + new String(buffer.array()));

                }

                //手动从集合中移动当前的selectionKey, 防止重复操作
                keyIterator.remove();

            }

        }

    }
}
```

### Java动态代理

[动态代理](https://snailclimb.gitee.io/javaguide/#/docs/java/basis/%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3?id=_3-%e5%8a%a8%e6%80%81%e4%bb%a3%e7%90%86)



## Java容器

------

看guide哥的容器总结足矣！

[JavaGuide容器](https://snailclimb.gitee.io/javaguide/#/?id=%e5%ae%b9%e5%99%a8)





## Java并发

------

### synchronized

偏向锁，轻量级锁，重量级锁，自旋锁，锁消除，锁粗化

[锁升级](https://blog.csdn.net/tongdanping/article/details/79647337)



### CAS

[CAS介绍](https://www.jianshu.com/p/ae25eb3cfb5d)

ABA问题：线程1预期值为A，这时线程2将值更改为B，然后又更改为A。然后线程1再去看值仍为A，但其实这个值已经被改了两次！



### lock

```java
public class Test {
    private ArrayList<Integer> arrayList = new ArrayList<Integer>();
    private Lock lock = new ReentrantLock();    //注意这个地方
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
    }  
     
    public void insert(Thread thread) {
        lock.lock();
        try {
            System.out.println(thread.getName()+"得到了锁");
            for(int i=0;i<5;i++) {
                arrayList.add(i);
            }
        } catch (Exception e) {
            // TODO: handle exception
        }finally {
            System.out.println(thread.getName()+"释放了锁");
            lock.unlock();
        }
    }
}
```

- Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现
- synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁



### volatile

> 内存可见性

Java内存模型，每个线程都会又一个本地内存来存储共享变量副本（本地内存使用寄存器，CPU缓存等高速缓存），线程所共享的区域为主存。使用volatile关键字，线程读，首先让本地内存从主存中读取最新值，然后在读取本地内存的值。线程写，先写入本地内存，然后立刻让本地更新的主存。



> 禁止指令重排

`uniqueInstance = new Singleton();` 这段代码其实是分为三步执行：

1. 为 `uniqueInstance` 分配内存空间
2. 初始化 `uniqueInstance`
3. 将 `uniqueInstance` 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 `getUniqueInstance`() 后发现 `uniqueInstance` 不为空，因此返回 `uniqueInstance`，但此时 `uniqueInstance` 还未被初始化。



### ThreadLocal

[ThreadLocal](https://snailclimb.gitee.io/javaguide/#/docs/java/multi-thread/2020%E6%9C%80%E6%96%B0Java%E5%B9%B6%E5%8F%91%E8%BF%9B%E9%98%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93?id=_3-threadlocal)



### 线程池

[线程池](https://snailclimb.gitee.io/javaguide/#/./docs/java/multi-thread/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93?id=%e4%b8%80-%e4%bd%bf%e7%94%a8%e7%ba%bf%e7%a8%8b%e6%b1%a0%e7%9a%84%e5%a5%bd%e5%a4%84)



### AQS

[AQS](https://snailclimb.gitee.io/javaguide/#/docs/java/multi-thread/AQS%E5%8E%9F%E7%90%86%E4%BB%A5%E5%8F%8AAQS%E5%90%8C%E6%AD%A5%E7%BB%84%E4%BB%B6%E6%80%BB%E7%BB%93?id=_1-aqs-%e7%ae%80%e5%8d%95%e4%bb%8b%e7%bb%8d)





## JVM

------

[JVM](https://snailclimb.gitee.io/javaguide/#/?id=jvm-%e5%bf%85%e7%9c%8b-1)



