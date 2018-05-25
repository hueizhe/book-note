## ThreadLocal

ThreadLocal是一个为线程提供线程局部变量的工具类。它的思想也十分简单，就是为线程提供一个线程私有的变量副本，这样多个线程都可以随意更改自己线程局部的变量，不会影响到其他线程。不过需要注意的是，ThreadLocal提供的只是一个浅拷贝，如果变量是一个引用类型，那么就要考虑它内部的状态是否会被改变，想要解决这个问题可以通过重写ThreadLocal的initialValue()函数来自己实现深拷贝，建议在使用ThreadLocal时一开始就重写该函数。

ThreadLocal与像synchronized这样的锁机制是不同的。

首先，它们的应用场景与实现思路就不一样，锁更强调的是如何同步多个线程去正确地共享一个变量，ThreadLocal则是为了解决同一个变量如何不被多个线程共享。

从性能开销的角度上来讲，如果锁机制是用时间换空间的话，那么ThreadLocal就是用空间换时间。

ThreadLocal中含有一个叫做ThreadLocalMap的内部类，该类为一个采用线性探测法实现的HashMap。它的key为ThreadLocal对象而且还使用了WeakReference，ThreadLocalMap正是用来存储变量副本的。

~~~java
/**
 * ThreadLocalMap is a customized hash map suitable only for
 * maintaining thread local values. No operations are exported
 * outside of the ThreadLocal class. The class is package private to
 * allow declaration of fields in class Thread.  To help deal with
 * very large and long-lived usages, the hash table entries use
 * WeakReferences for keys. However, since reference queues are not
 * used, stale entries are guaranteed to be removed only when
 * the table starts running out of space.
 */
static class ThreadLocalMap {
    /**
     * The entries in this hash map extend WeakReference, using
     * its main ref field as the key (which is always a
     * ThreadLocal object).  Note that null keys (i.e. entry.get()
     * == null) mean that the key is no longer referenced, so the
     * entry can be expunged from table.  Such entries are referred to
     * as "stale entries" in the code that follows.
     */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;
 
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    ....
}
~~~
ThreadLocal中只含有三个成员变量，这三个变量都是与ThreadLocalMap的hash策略相关的。

~~~java
/**
 * ThreadLocals rely on per-thread linear-probe hash maps attached
 * to each thread (Thread.threadLocals and
 * inheritableThreadLocals).  The ThreadLocal objects act as keys,
 * searched via threadLocalHashCode.  This is a custom hash code
 * (useful only within ThreadLocalMaps) that eliminates collisions
 * in the common case where consecutively constructed ThreadLocals
 * are used by the same threads, while remaining well-behaved in
 * less common cases.
 */
private final int threadLocalHashCode = nextHashCode();
 
/**
 * The next hash code to be given out. Updated atomically. Starts at
 * zero.
 */
private static AtomicInteger nextHashCode =
    new AtomicInteger();
 
/**
 * The difference between successively generated hash codes - turns
 * implicit sequential thread-local IDs into near-optimally spread
 * multiplicative hash values for power-of-two-sized tables.
 */
private static final int HASH_INCREMENT = 0x61c88647;
 
/**
 * Returns the next hash code.
 */
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
~~~
唯一的实例变量threadLocalHashCode是用来进行寻址的hashcode，它由函数nextHashCode()生成，该函数简单地通过一个增量HASH_INCREMENT来生成hashcode。至于为什么这个增量为0x61c88647，主要是因为ThreadLocalMap的初始大小为16，每次扩容都会为原来的2倍，这样它的容量永远为2的n次方，该增量选为0x61c88647也是为了尽可能均匀地分布，减少碰撞冲突。

~~~java
/**
 * The initial capacity -- MUST be a power of two.
 */
private static final int INITIAL_CAPACITY = 16;    
 
/**
 * Construct a new map initially containing (firstKey, firstValue).
 * ThreadLocalMaps are constructed lazily, so we only create
 * one when we have at least one entry to put in it.
 */
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
~~~
要获得当前线程私有的变量副本需要调用get()函数。首先，它会调用getMap()函数去获得当前线程的ThreadLocalMap，这个函数需要接收当前线程的实例作为参数。如果得到的ThreadLocalMap为null，那么就去调用setInitialValue()函数来进行初始化，如果不为null，就通过map来获得变量副本并返回。

setInitialValue()函数会去先调用initialValue()函数来生成初始值，该函数默认返回null，我们可以通过重写这个函数来返回我们想要在ThreadLocal中维护的变量。之后，去调用getMap()函数获得ThreadLocalMap，如果该map已经存在，那么就用新获得value去覆盖旧值，否则就调用createMap()函数来创建新的map。

~~~java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
 
/**
 * Variant of set() to establish initialValue. Used instead
 * of set() in case user has overridden the set() method.
 *
 * @return the initial value
 */
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
 
protected T initialValue() {
    return null;
}
~~~
ThreadLocal的set()与remove()函数要比get()的实现还要简单，都只是通过getMap()来获得ThreadLocalMap然后对其进行操作。

~~~java
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
 
/**
 * Removes the current thread's value for this thread-local
 * variable.  If this thread-local variable is subsequently
 * {@linkplain #get read} by the current thread, its value will be
 * reinitialized by invoking its {@link #initialValue} method,
 * unless its value is {@linkplain #set set} by the current thread
 * in the interim.  This may result in multiple invocations of the
 * {@code initialValue} method in the current thread.
 *
 * @since 1.5
 */
 public void remove() {
     ThreadLocalMap m = getMap(Thread.currentThread());
     if (m != null)
         m.remove(this);
 }
 ~~~
getMap()函数与createMap()函数的实现也十分简单，但是通过观察这两个函数可以发现一个秘密：ThreadLocalMap是存放在Thread中的。

~~~java
/**
 * Get the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param  t the current thread
 * @return the map
 */
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
 
/**
 * Create the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param t the current thread
 * @param firstValue value for the initial entry of the map
 */
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
 
// Thread中的源码
 
/**
  ThreadLocal values pertaining to this thread. This map is maintained
  by the ThreadLocal class. 
 */
ThreadLocal.ThreadLocalMap threadLocals = null;
 
/**
 * InheritableThreadLocal values pertaining to this thread. This map is
 * maintained by the InheritableThreadLocal class.
 */
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
~~~
仔细想想其实就能够理解这种设计的思想。有一种普遍的方法是通过一个全局的线程安全的Map来存储各个线程的变量副本，但是这种做法已经完全违背了ThreadLocal的本意，设计ThreadLocal的初衷就是为了避免多个线程去并发访问同一个对象，尽管它是线程安全的。而在每个Thread中存放与它关联的ThreadLocalMap是完全符合ThreadLocal的思想的，当想要对线程局部变量进行操作时，只需要把Thread作为key来获得Thread中的ThreadLocalMap即可。这种设计相比采用一个全局Map的方法会多占用很多内存空间，但也因此不需要额外的采取锁等线程同步方法而节省了时间上的消耗。

## ThreadLocal中的内存泄漏

我们要考虑一种会发生内存泄漏的情况，如果ThreadLocal被设置为null后，而且没有任何强引用指向它，根据垃圾回收的可达性分析算法，ThreadLocal将会被回收。这样一来，ThreadLocalMap中就会含有key为null的Entry，而且ThreadLocalMap是在Thread中的，只要线程迟迟不结束，这些无法访问到的value会形成内存泄漏。为了解决这个问题，ThreadLocalMap中的getEntry()、set()和remove()函数都会清理key为null的Entry，以下面的getEntry()函数的源码为例。

~~~java
/**
 * Get the entry associated with key.  This method
 * itself handles only the fast path: a direct hit of existing
 * key. It otherwise relays to getEntryAfterMiss.  This is
 * designed to maximize performance for direct hits, in part
 * by making this method readily inlinable.
 *
 * @param  key the thread local object
 * @return the entry associated with key, or null if no such
 */
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
 
/**
 * Version of getEntry method for use when key is not found in
 * its direct hash slot.
 *
 * @param  key the thread local object
 * @param  i the table index for key's hash code
 * @param  e the entry at table[i]
 * @return the entry associated with key, or null if no such
 */
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;
 
    // 清理key为null的Entry
    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
~~~
在上文中我们发现了ThreadLocalMap的key是一个弱引用，那么为什么使用弱引用呢？使用强引用key与弱引用key的差别如下：

- 强引用key：ThreadLocal被设置为null，由于ThreadLocalMap持有ThreadLocal的强引用，如果不手动删除，那么ThreadLocal将不会回收，产生内存泄漏。

- 弱引用key：ThreadLocal被设置为null，由于ThreadLocalMap持有ThreadLocal的弱引用，即便不手动删除，ThreadLocal仍会被回收，ThreadLocalMap在之后调用set()、getEntry()和remove()函数时会清除所有key为null的Entry。

但要注意的是，ThreadLocalMap仅仅含有这些被动措施来补救内存泄漏问题。如果你在之后没有调用ThreadLocalMap的set()、getEntry()和remove()函数的话，那么仍然会存在内存泄漏问题。
在使用线程池的情况下，如果不及时进行清理，内存泄漏问题事小，甚至还会产生程序逻辑上的问题。

所以，**为了安全地使用ThreadLocal，必须要像每次使用完锁就解锁一样，在每次使用完ThreadLocal后都要调用remove()来清理无用的Entry。**