## 进程线程

一个程序就是一个进程，进程会占用内存空间，线程是进程的一个实体，一个进程可以拥有多个线程。

多线程是多任务的一种特别的形式，但多线程使用了更小的资源开销。

## 线程的生命周期

![img](https://www.runoob.com/wp-content/uploads/2014/01/java-thread.jpg)

![](/Users/weizhile/Documents/my_notebook/java并发/1640778014-cXclgm-image.png)

## 创建一个线程

Java 提供了三种创建线程的方法：

- 通过实现 Runnable 接口；
- 通过继承 Thread 类本身；
- 通过 Callable 和 Future 创建线程。

<<<<<<< HEAD
> 线程与任务分离 — Thread + 实现 Callable 接口
> 虽然 Runnable 挺不错的，但是仍然有个缺点，那就是没办法获取任务的执行结果，因为它的 run 方法返回值是 void。
>
> 这样，对于需要获取任务执行结果的线程来说，Callable 就成为了一个完美的选择。
>
> 
>
> 和 Runnbale 比起来，Callable 不过就是把 run 改成了 call。当然，最重要的是！和 void run 不同，这个 call 方法是拥有返回值的，而且能够抛出异常。
>
> 这样，一个很自然的想法，就是把 Callable 作为任务对象传给 Thread，然后 Thread 重写 call 方法就完事儿。
>
> But，遗憾的是，Thread 类的构造函数里并不接收 Callable 类型的参数。
>
> 所以，我们需要把 Callable 包装一下，包装成 Runnable 类型，这样就能传给 Thread 构造函数了。
>
> 为此，FutureTask 成为了最好的选择。
>

```java
class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        // 要执行的任务
        ......
        return 100;
    }
}
// 将 Callable 包装成 FutureTask，FutureTask也是一种Runnable
MyCallable callable = new MyCallable();
FutureTask<Integer> task = new FutureTask<>(callable);
// 创建线程对象
Thread t3 = new Thread(task);
```

当线程运行起来后，可以通过 FutureTask 的 get 方法获取任务运行结果：

`Integer result = task.get();`
不过，需要注意的是，get 方法会阻塞住当前调用这个方法的线程。比如说我们在主线程中调用了 get 方法去获取 t3 线程的任务运行结果，那么只有这个 call 方法成功返回了，主线程才能够继续往下执行。

换句话说，如果 call 方法一直得不到结果，那么主线程也就一直无法向下运行。

## 启动线程

这里涉及一道经典的面试题，即为什么使用 start 启动线程，而不使用 run 方法启动线程？

使用 run 方法启动线程看起来好像并没啥问题，对吧，run 方法内定义了要执行的任务，调用 run 方法不就执行了这个任务了？

这确实没错，任务确实能够被正确执行，但是并不是以多线程的方式，当我们使用 t1.run() 的时候，程序仍然是在创建 t1 线程的 main 线程下运行的，并没有创建出一个新的 t1 线程。

=======
>>>>>>> 9b615f924c671c794f7ec87dadbd20cee687e50d
## 线程安全

诚然，一个程序顺序的运行多个线程本身是没有问题的，但是如果多个线程同时访问了某个共享资源，就可能会发生不可预知的现象，也就是我们常说的线程安全问题，要了解这些问题产生的根本原因，我们就需要去深刻的了解 Java 内存模型（Java Memory Model，JMM）。

为此，我们会学习到和线程安全息息相关的三大性质：

1）原子性：一个操作是不可中断的，要么全部执行成功要么全部执行失败（也可以说是提供互斥访问，同一时刻只能有一个线程对数据进行操作）

2）可见性：当一个线程修改了共享变量后，其他线程能够立即得知这个修改

3）有序性：编译器和处理器为了优化程序性能，会对指令序列进行重新排序。由于重排序的存在，可能导致多线程环境下程序运行结果出错的问题。

### 线程的底层运行原理![](E:\MarkDown\my_notebook\java并发\1640672008-xjjChb-image.png)

1）Java 堆（Java Heap）是 Java 虚拟机所管理的内存中最大的一块，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。这一点在 Java 虚拟机规范中的描述是：所有的对象实例以及数组都要在堆上分配。

2）方法区（Method Area）与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

很多人习惯的把方法区称为永久代（Permanent Generation），但实际上这两者并不等价。通俗来说，方法区是一种规范，而永久代是 HotSpot 虚拟机实现这个规范的一种手段，对于其他虚拟机（比如 BEA JRockit、IBM J9 等）来说是不存在永久代的概念的。

另外，对于 HotSpot 虚拟机来说，它在 JDK 8 中完全废弃了永久代的概念，改用与 JRockit、J9 一样在本地内存中实现的元空间（Meta-space）来代替，把 JDK 7 中永久代还剩余的内容（主要是类型信息）全部移到元空间中。

再来看看线程私有的三个区域：

1）虚拟机栈（Java Virtual Machine Stacks）其实是由一个一个的 栈帧（Stack Frame） 组成的，一个栈帧描述的就是一个 Java 方法执行的内存模型。也就是说每个方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法的返回地址等信息。

![](E:\MarkDown\my_notebook\java并发\1640672024-zYWIBJ-image.png)

每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程，当然，出栈的顺序自然是遵守栈的后进先出原则的。

栈帧的概念在接下来的原理解析部分非常重要，各位务必搞懂哈。

2）本地方法栈（Native Method Stack）和上面我们所说的虚拟机栈作用基本一样，区别只不过是本地方法栈为虚拟机使用到的 Native 方法服务，而虚拟机栈为虚拟机执行 Java 方法（也就是字节码）服务。

这里解释一下 Native 方法的概念，其实不仅 Java，很多语言中都有这个概念。

"A native method is a Java method whose implementation is provided by non-java code."

就是说一个 Native 方法其实就是一个接口，但是它的具体实现是在外部由非 Java 语言写的。所以同一个 Native 方法，如果用不同的虚拟机去调用它，那么得到的结果和运行效率可能是不一样的，因为不同的虚拟机对于某个 Native 方法都有自己的实现，比如 Object 类的 hashCode 方法。

这使得 Java 程序能够超越 Java 运行时的界限，有效地扩充了 JVM。

3）程序计数器（Program Counter Register）是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

由于 Java 虚拟机的多线程是通过轮流分配 CPU 时间片的方式来实现的，因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器。

## Java的内存模型

#### 物理内存模型

**物理机对并发的处理方案对虚拟机的实现也有相当大的参考意义**

在硬件层面上，现代计算机系统都会在内存与 CPU 之间加入一层或多层读写速度尽可能接近 CPU 运算速度的高速缓存来作为缓冲。

将运算需要使用的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中，这样处理器就无须等待缓慢的内存读写了。

为此，这不可避免的带来了一个新的问题：**缓存一致性（Cache Coherence）。**

就是说当多个 CPU 的运算任务都涉及同一块主内存区域时，将可能导致各自的缓存数据不一致。如果真的发生这种情况，那同步回到主内存时该以谁的缓存数据为准呢？

为了解决一致性的问题，需要各个 CPU 访问缓存时都遵循一些协议，在读写时要根据协议来进行操作。于是，我们引出了内存模型的概念。

![](E:\MarkDown\my_notebook\java并发\1640777399-yWclFk-image.png)

在物理机层面，内存模型可以理解为在特定的操作协议下，对特定的内存或高速缓存进行读写访问的过程抽象。

显然，不同架构的物理机器可以拥有不一样的内存模型，而 **Java 虚拟机也拥有自己的内存模型，称为 Java 内存模型（Java Memory Model，JMM），其目的就是为了屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果**。

> 当然了，JMM 与这里我们介绍的物理机的内存模型具有高度的可类比性。

#### Java 内存模型

JMM 规定了所有的变量都存储在**主内存**（Main Memory）中，每条线程还有自己的**工作内存**（Working Memory）。

线程的工作内存中保存了被该线程使用的变量的主内存副本，线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行，而不能直接读写主内存中的数据。

![](E:\MarkDown\my_notebook\java并发\1640777418-ocMlIT-image.png)

此处的主内存可以与前面所说的物理机的主内存类比，当然，实际上它仅是虚拟机内存的一部分，工作内存可与前面讲的高速缓存类比。

## 原子性

**Java 虚拟机实现时必须保证下面提及的每一种操作都是原子的、不可再分的**。

> 举个经典的简单例子，银行转账，A 像 B 转账 100 元。转账这个操作其实包含两个离散的步骤：
>
> 步骤 1：A 账户减去 100
> 步骤 2：B 账户增加 100
> 我们要求转账这个操作是原子性的，也就是说步骤 1 和步骤 2 是顺续执行且不可被打断的，要么全部执行成功、要么执行失败。

JMM 定义的 8 种原子操作具体：

**lock（锁定）：**作用于主内存的变量，它把一个变量标识为一条线程独占的状态。
**unlock（解锁）：**作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
**read（读取）：**作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。
**load（载入）：**作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
**use（使用）：**作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
**assign（赋值）：**作用于工作内存的变量，它把一个从执行引擎接收的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
**store（存储）：**作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用。
**write（写入）：**作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量处。



java中i++,i=j这样的普遍使用的操作语句都不是原子性的,详情可见文件夹下atomic包里的代码

#### 如何保证原子性

**那么，如何实现原子操作，也就是如何保证原子性呢？**

对于这个问题，其实在处理器和 Java 编程语言层面，它们都提供了一些有效的措施，比如处理器提供了总线锁和缓存锁，**Java 提供了锁和循环 CAS 的方式**，这里我们简单解释下 Java 保证原子性的措施。

由 Java 内存模型来直接保证的原子性变量操作包括 read、load、assign、use、store 和 write 这 6 个，我们大致可以认为，基本数据类型的访问、读写都是具备原子性的

## 可见性

**何为可见性**？就是指当一个线程修改了共享变量的值时，其他线程能够**立即**得知这个修改。

**线程 A 在向线程 B 的通信过程必须要经过主内存**。

#### 如何保证可见性

各位可能脱口而出使用 **volatile 关键字**修饰共享变量，但除了这个，容易被大家忽略的是，其实 **sunchronized** 和 **final** 这俩关键字也能保证可见性。

## 有序性

#### 什么是有序性

为了使 CPU 内部的运算单元能尽量被充分利用，CPU 可能会对输入代码进行乱序执行优化，CPU 会在计算之后将乱序执行的结果重组，保证该结果与顺序执行的结果是一致的。与之类似的，Java 的编译器也有这样的一种优化手段：**指令重排序**（Instruction Reorder）。

>  那么，既然能够优化性能，重排序可以没有限制的被使用吗？

当然不，在重排序的时候，CPU 和编译器都需要遵守一个规矩，这个规矩就是 **as-if-serial** 语义：不管怎么重排序，**单线程**环境下程序的执行结果不能被改变。

为了遵守 as-if-serial 语义，CPU 和编译器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。

那么这里，我们又引出了 “**数据依赖性**” 的概念。

如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性。

数据依赖性分为三种类型：写后读、写后写、读后写

**CPU 和 Java 编译器为了优化程序性能，会自发地对指令序列进行重新排序。在多线程的环境下，由于重排序的存在，就可能导致程序运行结果出现错误**

单线程内就不用担心会有这个影响啦

#### 如何保证有序性

Java 语言提供了 **volatile** 和 **synchronized** 两个关键字来保证线程之间操作的有序性。

- volatile 本身除了保证可见性的语义外，还包含了禁止指令重排序的语义，所以天生就具有保证有序性的功能。

- 而 synchronized 保证有序性的理论支撑，仍然是 JMM 规定在执行 8 种基本原子操作时必须满足的一系列规则中的某一个提供的：

  - 一个变量在同一个时刻只允许一条线程对其进行 lock 操作

    >  这个规则决定了持有同一个锁的两个 synchronized 同步块只能串行地进入。

通俗来说，synchronized 通过排他锁的方式保证了同一时间内，被 synchronized 修饰的代码是单线程执行的。所以，这就满足了 as-if-serial 语义的一个关键前提，那就是单线程，这样，有了 as-if-serial 语义的保证，单线程的有序性也就得到保障了。

But，遗憾的是，如果仅仅依靠这俩个关键字来保证有序性的话，编码将会变得非常繁琐，为此，**Happens-before** 原则应运而生。

### Happens-before 原则

Happens-before 是 JMM 的灵魂，它是判断数据是否存在竞争，线程是否安全的非常有用的手段。

Happens-before 直译为 “先行发生”，《JSR-133：Java Memory Model and Thread Specification》对 Happens-before 关系的定义如下：

> 1）如果一个操作 Happens-before 另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
>
> 2）两个操作之间存在 Happens-before 关系，并不意味着 Java 平台的具体实现必须要按照 Happens-before 关系指定的顺序来执行。如果重排序之后的执行结果，与按 Happens-before 关系来执行的结果一致，那么这种重排序并不非法（也就是说，JMM 允许这种重排序）



