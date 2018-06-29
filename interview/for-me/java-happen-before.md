## Java 使用 happen-before 规则实现共享变量的同步操作

### 前言

熟悉 Java 并发编程的都知道，JMM(Java 内存模型) 中的 happen-before(简称 hb)规则，该规则定义了 Java 多线程操作的有序性和可见性，防止了编译器重排序对程序结果的影响。按照官方的说法：

当一个变量被多个线程读取并且至少被一个线程写入时，如果读操作和写操作没有 HB 关系，则会产生数据竞争问题。 要想保证操作 B 的线程看到操作 A 的结果（无论 A 和 B 是否在一个线程），那么在 A 和 B 之间必须满足 HB 原则，如果没有，将有可能导致重排序。 当缺少 HB 关系时，就可能出现重排序问题。


### HB 有哪些规则？
这个大家都非常熟悉了应该，大部分书籍和文章都会介绍，这里稍微回顾一下：

- 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作；
- 锁定规则：在监视器锁上的解锁操作必须在同一个监视器上的加锁操作之前执行。
- volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作；
- **传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C**；
- 线程启动规则：Thread对象的start()方法先行发生于此线程的每一个动作；
- 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生；
- 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行；
- 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始；

其中，传递规则我加粗了，这个规则至关重要。如何熟练的使用传递规则是实现同步的关键。

然后，再换个角度解释 HB：当一个操作 A HB 操作 B，那么，操作 A 对共享变量的操作结果对操作 B 都是可见的。

同时，如果 操作 B HB 操作 C，那么，操作 A 对共享变量的操作结果对操作 B 都是可见的。

而实现可见性的原理则是 cache protocol 和 memory barrier。通过缓存一致性协议和内存屏障实现可见性。

### 如何实现同步？
在 Doug Lea 著作 《Java Concurrency in Practice》中，有下面的描述：
>
>

书中提到：通过组合 hb 的一些规则，可以实现对某个未被锁保护变量的可见性。

但由于这个技术对语句的顺序很敏感，因此容易出错。

楼主接下来，将演示如何通过 volatile 规则和程序次序规则实现对一个变量同步。

来一个熟悉的例子：

~~~java
class ThreadPrintDemo{

  static int num = 0;
  static volatile boolean flag = false;

  public static void main(String[] args){

    Thread t1 = new Thread(() -> {
      for (; 100 > num; ) {
        if (!flag && (num == 0 || ++num % 2 == 0)) {
          System.out.println(num);
          flag = true;
        }
      }
    }
    );

    Thread t2 = new Thread(() -> {
      for (; 100 > num; ) {
        if (flag && (++num % 2 != 0)) {
          System.out.println(num);
          flag = false;
        }
      }
    }
    );

    t1.start();
    t2.start();
  }
}

~~~
这段代码的作用是两个线程间隔打印出 0 – 100 的数字。

熟悉并发编程的同学肯定要说了，这个 num 变量没有使用 volatile，会有可见性问题，即：t1 线程更新了 num，t2 线程无法感知。

哈哈，楼主刚开始也是这么认为的，但最近通过研究 HB 规则，我发现，去掉 num 的 volatile 修饰也是可以的。

[](pic/happen-before.png)

我们分析这个图：

首先，红色和黄色表示不同的线程操作。
红色线程对 num 变量做 ++，然后修改了 volatile 变量，这个是符合 程序次序规则的。也就是 1 HB 2.
红色线程对 volatile 的写 HB 黄色线程对 volatile 的读，也就是 2 HB 3.
黄色线程读取 volatile 变量，然后对 num 变量做 ++，符合 程序次序规则，也就是 3 HB 4.
根据传递性规则，1 肯定 HB 4. 所以，1 的修改对 4来说都是可见的。
注意：HB 规则保证上一个操作的结果对下一个操作都是可见的。

所以，上面的小程序中，线程 A 对 num 的修改，线程 B 是完全感知的 —— 即使 num 没有使用 volatile 修饰。

这样，我们就借助 HB 原则实现了对一个变量的同步操作，也就是在多线程环境中，保证了并发修改共享变量的安全性。并且没有对这个变量使用 Java 的原语：volatile 和 synchronized 和 CAS（假设算的话）。

这可能看起来不安全（实际上安全），也好像不太容易理解。因为这一切都是 HB 底层的 cache protocol 和 memory barrier 实现的。

### 其他规则实现同步

1. 利用线程终结规则实现：
~~~java
static int a = 1;

  public static void main(String[] args){
    Thread tb = new Thread(() -> {
      a = 2;
    });
    Thread ta = new Thread(() -> {
      try {
        tb.join();
      } catch (InterruptedException e) {
        //NO
      }
      System.out.println(a);
    });

    ta.start();
    tb.start();
  }
~~~

2. 利用线程 start 规则实现：
~~~java
static int a = 1;

  public static void main(String[] args){
    Thread tb = new Thread(() -> {
      System.out.println(a);
    });
    Thread ta = new Thread(() -> {
      tb.start();
      a = 2;
    });

    ta.start();
  }
~~~

这两个操作，也可以保证变量 a 的可见性。

确实有点颠覆之前的观念。之前的观念中，如果一个变量没有被 volatile 修饰或 final 修饰，那么他在多线程下的读写肯定是不安全的 —— 因为会有缓存，导致读取到的不是最新的。

然而，通过借助 HB，我们可以实现。

## 总结
虽然本文标题是通过 happen-before 实现对共享变量的同步操作，但主要目的还是更深刻的理解 happen-before，理解他的 happen-before 概念其实就是保证多线程环境中，上一个操作对下一个操作的有序性和操作结果的可见性。

同时，通过灵活的使用传递性规则，再对规则进行组合，就可以将两个线程进行同步 —— 实现指定的共享变量不使用原语也可以保证可见性。虽然这好像不是很易读，但也是一种尝试。

关于如何组合使用规则实现同步，Doug Lea 在 JUC 中给出了实践。

例如老版本的 FutureTask 的内部类 Sync（已消失），通过 tryReleaseShared 方法修改 volatile 变量，tryAcquireShared 读取 volatile 变量，这是利用了 volatile 规则；

通过在 tryReleaseShared 之前设置非 volatile 的 result 变量，然后在 tryAcquireShared 之后读取 result 变量，这是利用了程序次序规则。

从而保证 result 变量的可见性。和我们的第一个例子类似：利用程序次序规则和 volatile 规则实现普通变量可见性。

 

而 Doug Lea 自己也说了，这个“借助”技术非常容易出错，要谨慎使用。但在某些情况下，这种“借助”是非常合理的。

实际上，BlockingQueue 也是“借助”了 happen-before 的规则。还记得 unlock 规则吗？当 unlock 发生后，内部元素一定是可见的。

而类库中还有其他的操作也“借助”了 happen-before 原则：并发容器，CountDownLatch，Semaphore，Future，Executor，CyclicBarrier，Exchanger 等。

 

### 总而言之，言而总之：

happen-before 原则是 JMM 的核心所在，只有满足了 hb 原则才能保证有序性和可见性，否则编译器将会对代码重排序。hb 甚至将 lock 和 volatile 也定义了规则。

通过适当的对 hb 规则的组合，可以实现对普通共享变量的正确使用。