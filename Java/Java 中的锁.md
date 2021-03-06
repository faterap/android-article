# Java 中的锁

## 锁的含义

### 性质

- **内存可见性**：同步快的可见性是由“如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值”、“对一个变量执行unlock操作之前，必须先把此变量同步回主内存中（执行store和write操作）”这两条规则获得的。
- **操作原子性**：持有同一个锁的两个同步块只能串行地进入

### 锁内存含义

- 当线程释放锁时，JVM 会把该线程对应的本地内存中的共享变量刷新到主内存中
- 当线程获取锁时，JVM 会把该线程对应的本地内存置为无效。从而使得被监视器保护的临界区代码必须从主内存中读取共享变量

### 锁释放和锁获取内存含义

- 线程A释放一个锁，实质上是线程A向接下来将要获取这个锁的某个线程发出了（线程A对共享变量所做修改的）消息。
- 线程B获取一个锁，实质上是线程B接收了之前某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息。
- 线程A释放锁，随后线程B获取这个锁，这个过程实质上是线程A通过主内存向线程B发送消息

![img](https://pic1.zhimg.com/80/v2-9b408e5de9536f47d32db62bb269a9a8_hd.jpg)

## 可重入锁

广义上的可重入锁指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁（前提得是同一个对象或者class），这样的锁就叫做可重入锁。

## Synchronized 锁

### 应用方式

- 修饰实例方法，作用于当前实例加锁。
- 修饰静态方法，作用于当前类对象加锁。
- 修饰代码块，指定加锁对象，对给定对象加锁。

### 底层原理

synchronized的底层是使用操作系统的mutex lock实现的。

### 内容

synchronized用的锁是存在Java对象头里的。

JVM基于进入和退出Monitor对象来实现方法同步和代码块同步。代码块同步是使用monitorenter和monitorexit指令实现的，monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处。任何对象都有一个monitor与之关联，当且一个monitor被持有后，它将处于锁定状态。

根据虚拟机规范的要求，在执行monitorenter指令时，首先要去尝试获取对象的锁，如果这个对象没被锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加1；相应地，在执行monitorexit指令时会将锁计数器减1，当计数器被减到0时，锁就释放了。如果获取对象锁失败了，那当前线程就要阻塞等待，直到对象锁被另一个线程释放为止。

注意两点：

1、synchronized同步块对同一条线程来说是**可重入的**，不会出现自己把自己锁死的问题；

2、同步块在已进入的线程执行完之前，会阻塞后面其他线程的进入。

## Mutex lock

监视器锁（Monitor）本质是依赖于底层的操作系统的Mutex Lock（互斥锁）来实现的。每个对象都对应于一个可称为" 互斥锁" 的标记，这个标记用来保证在任一时刻，只能有一个线程访问该对象。

互斥锁：**用于保护临界区，确保同一时间只有一个线程访问数据。对共享资源的访问**，先对互斥量进行加锁，如果互斥量已经上锁，调用线程会阻塞，直到互斥量被解锁。在完成了对共享资源的访问后，要对互斥量进行解锁。

![img](https://pic4.zhimg.com/80/v2-ba0c5a802bc7c45d3add8214ad6f1eaf_hd.jpg)

## Java 对象头

![img](https://user-gold-cdn.xitu.io/2018/7/29/164e3637df80c2a2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 偏向锁

大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得。偏向锁是为了在只有一个线程执行同步块时提高性能。

当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁。引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次CAS原子指令，而偏向锁只需要在置换ThreadID的时候依赖一次CAS原子指令（由于一旦出现多线程竞争的情况就必须撤销偏向锁，所以偏向锁的撤销操作的性能损耗必须小于节省下来的CAS原子指令的性能消耗）。

## 轻量级锁

轻量级锁是为了在线程近乎交替执行同步块时提高性能。

## 重量级锁

指向互斥量（mutex），底层通过操作系统的mutex lock实现。等待锁的线程会被阻塞，由于Linux下Java线程与操作系统内核态线程一一映射，所以**涉及到用户态和内核态的切换**、操作系统内核态中的线程的阻塞和恢复。





------

[]: https://zhuanlan.zhihu.com/p/29866981	"Java synchronized原理总结"
[]: https://juejin.im/post/5b4eec7df265da0fa00a118f#heading-13	"啃碎并发（七）：深入分析Synchronized原理"
[]: https://blog.csdn.net/javazejian/article/details/72828483#synchronized%E6%96%B9%E6%B3%95%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86	"深入理解Java并发之synchronized实现原理"

[]: https://tech.meituan.com/2018/11/15/java-lock.html	"不可不说的Java“锁”事"

