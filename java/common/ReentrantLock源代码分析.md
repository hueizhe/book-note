## ReentrantLock源代码分析

## 什么是AQS

AQS是JUC里并发控制一个很重要的基础，他提供了一个独占和共享访问控制某个状态的工具，JUC里的锁、信号量、门阀都是基于AQS实现的，单独去看AQS的代码很难理解，而且很难理解其设计之精妙，笔者计划依次分析锁(ReentrantLock)、门阀(CountDownLatch)、信号量(Semaphor)、读写锁(ReadWriteLock)来解析AQS的设计细节。


如果是我怎么实现
在看ReentrantLock代码之前，我们试想一下如果是我们自己去实现要如何做?忽略可重入、公平等锁的特性，需求具体来说有以下两点：
1. 多个线程同时请求锁时只有一个线程会取得锁，其他线程进入队列等待
2. 锁释放时会通知其他等待队列中的线程去获取锁

所以如果我们自己去实现的话翻译成技术语言需要有以下几个基础设施
1. 决定谁获得锁的竞争机制，这个很容易想到用CAS去实现
2. 记录锁状态，这个可以用一个变量表示锁计数，一个变量表示当前获取锁的线程（不考虑可重入其实非必要）
3. 锁释放时通知队列中第一个线程并使其获得锁

看起来也不难对不对？让我们一起来看看道格李是怎么优雅地实现这些功能的吧

### 源代码分析
#### lock方法
我们先看lock的代码，忽略简单的嵌套调用，lock方法实际调用的是AQS的acquire方法：
~~~java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
~~~
再往下看tryAcquire方法，这个方法AQS里没有实现，直接抛出了异常，这么做是避免子类实现所有接口，我们看java.util.concurrent.locks.ReentrantLock.FairSync这个AQS子类的实现
~~~java
protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
// c=0 说明没有其他线程占有锁
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
// 队列中没有其他线程在等待锁，而且CAS把state设置成入参的值成功，这里是1（这里的CAS就是我
// 们前文提的并发竞争机制），则当前线程获取锁成功并将owner线程设置为当前线程
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
// 可重入设置，当前线程重复请求锁成功，只是增加请求锁的计数
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
~~~
再看acquire方法如果tryAcquire成功了就直接返回，不用执行后面的代码，如果tryAcquire失败了就执行acquireQueued(addWaiter(Node.EXCLUSIVE), arg))，我们先看addWaiter方法，这个方法是把当前请求放到队列中：

~~~java
private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);

// Try the fast path of enq; backup to full enq on failure
// 上面这个官方注释很直白，其实下面的enq方法里也执行了这段代码，但是这里先直接试一下看能
//  否插入成功
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
// CAS把tail设置成当前节点，如果成功的话就说明插入成功，直接返回node，失败说明有其他线程也
// 在尝试插入而且其他线程成功,如果是这样就继续执行enq方法
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }
~~~
继续看enq方法：

~~~java
private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
// 最开始head和tail都是空的，需要通过CAS做初始化，如果CAS失败，则循环重新检查tail
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
// head和tail不是空的，说明已经完成初始化，和addWaiter方法的上半段一样，CAS修改
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
~~~
将当前请求锁失败的节点插入到队列中之后还执行了acquireQueued方法，因为我们执行插入队列之后还没有阻塞当前线程呢：

~~~java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
/*
* 如果前置节点是head，说明当前节点是队列第一个等待的节点，这时去尝试获取锁，如果成功了则
* 获取锁成功。这里有的同学可能没看懂，不是刚尝试失败并插入队列了吗，咋又尝试获取锁？ 其实这*
* 里是个循环，其他刚被唤醒的线程也会执行到这个代码
*/
                if (p == head && tryAcquire(arg)) {
// 队首且获取锁成功，把当前节点设置成head，下一个节点成了等待队列的队首
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
/* shouldParkAfterFailedAcquire方法判断如果获取锁失败是否需要阻塞，如果需要的话就执行
*  parkAndCheckInterrupt方法，如果不需要就继续循环
*/
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
下面看一下shouldParkAfterFailedAcquire方法：

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
// 获取pred前置节点的等待状态
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
/* 前置节点状态是signal，那当前节点可以安全阻塞，因为前置节点承诺执行完之后会通知唤醒当前
* 节点
*/
            return true;
        if (ws > 0) {

            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
// 前置节点如果已经被取消了，则一直往前遍历直到前置节点不是取消状态，与此同时会修改链表关系
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
// 前置节点是0或者propagate状态，这里通过CAS把前置节点状态改成signal
// 这里不返回true让当前节点阻塞，而是返回false，目的是让调用者再check一下当前线程是否能
// 成功获取锁，失败的话再阻塞，这里说实话我也不是特别理解这么做的原因
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
~~~
假设前面一步返回true需要阻塞，则会调用parkAndCheckInterrupt进行阻塞

~~~java
private final boolean parkAndCheckInterrupt() {
// 阻塞当前线程，监事是当前sync对象
        LockSupport.park(this);
// 阻塞返回后，返回当前线程是否被中断
        return Thread.interrupted();
    }
park方法

public static void park(Object blocker) {
        Thread t = Thread.currentThread();
// 设置当前线程的监视器blocker
        setBlocker(t, blocker);
// 这里调用了native方法到JVM级别的阻塞机制阻塞当前线程
        UNSAFE.park(false, 0L);
// 阻塞结束后把blocker置空
        setBlocker(t, null);
    }
~~~
至此，一次lock的调用就完成了，总结来说：

调用tryAcquire方法尝试获取锁，获取成功的话修改state并直接返回true，获取失败的话把当前线程加到等待队列中
加到等待队列之后先检查前置节点状态是否是signal，如果是的话直接阻塞当前线程等待唤醒，如果不是的话判断是否是cancel状态，是cancel状态就往前遍历并把cancel状态的节点从队列中删除。如果状态是0或者propagate的话将其修改成signal
阻塞被唤醒之后如果是队首并且尝试获取锁成功就返回true，否则就继续执行前一步的代码进入阻塞
unlock方法
看完了lock方法再来看unlock方法，同样unlock方法调用的就是AQS的release方法
~~~java
public final boolean release(int arg) {
/*
 尝试释放锁如果失败，直接返回失败，如果成功并且head的状态不等于0就唤醒后面等待的节点
*/
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
~~~
我们先看tryRelease方法：
~~~java
protected final boolean tryRelease(int releases) {
// 释放后c的状态值
            int c = getState() - releases;
// 如果持有锁的线程不是当前线程，直接抛出异常
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
// 如果c==0，说明所有持有锁都释放完了，其他线程可以请求获取锁
                free = true;
                setExclusiveOwnerThread(null);
            }
// 这里只会有一个线程执行到这，不存在竞争，因此不需要CAS
            setState(c);
            return free;
        }
~~~
再看看unparkSuccessor方法：
~~~java
private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        if (ws < 0)
/*
如果状态小于0，把状态改成0，0是空的状态，因为node这个节点的线程释放了锁后续不需要做任何
操作，不需要这个标志位，即便CAS修改失败了也没关系，其实这里如果只是对于锁来说根本不需要CAS，因为这个方法只会被释放锁的线程访问，只不过unparkSuccessor这个方法是AQS里的方法就必须考虑到多个线程同时访问的情况（可能共享锁或者信号量这种）
*/
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
// 这段代码的作用是如果下一个节点为空或者下一个节点的状态>0（目前大于0就是取消状态）
// 则从tail节点开始遍历找到离当前节点最近的且waitStatus<=0（即非取消状态）的节点并唤醒
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
~~~
至此，一次unlock的调用完成了，总结来说：

1. 修改状态位
2. 唤醒排队的节点
3. 结合lock方法，被唤醒的节点会自动替换当前节点成为head

### 总结

总的来说，用AQS来实现ReentrantLock还是比较简单，因为互斥地访问，不会存在太多并发访问某个方法的场景，只需要处理好请求锁竞争和释放锁的过程就可以了。后面笔者会继续分析较为复杂的Semaphore信号量、CountDownLatch、ReadWriteLock