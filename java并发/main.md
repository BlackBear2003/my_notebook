## 进程线程

一个程序就是一个进程，进程会占用内存空间，线程是进程的一个实体，一个进程可以拥有多个线程。

多线程是多任务的一种特别的形式，但多线程使用了更小的资源开销。

## 线程的生命周期

![img](https://www.runoob.com/wp-content/uploads/2014/01/java-thread.jpg)

## 创建一个线程

Java 提供了三种创建线程的方法：

- 通过实现 Runnable 接口；
- 通过继承 Thread 类本身；
- 通过 Callable 和 Future 创建线程。