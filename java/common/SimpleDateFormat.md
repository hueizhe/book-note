## SimpleDateFormat是线程不安全的
> SimpleDateFormat是Java提供的一个格式化和解析日期的工具类，日常开发中应该经常会用到，但是由于它是线程不安全的，多线程公用一个SimpleDateFormat实例对日期进行解析或者格式化会导致程序出错，本节就讨论下它为何是线程不安全的，以及如何避免。


## 问题复现
为了复现该问题，编写如下代码：
~~~
public class TestSimpleDateFormat {
    //(1)创建单例实例
    static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static void main(String[] args) {
        //(2)创建多个线程，并启动
        for (int i = 0; i <10 ; ++i) {
            Thread thread = new Thread(new Runnable() {
                public void run() {
                    try {//(3)使用单例日期实例解析文本
                        System.out.println(sdf.parse("2017-12-13 15:17:27"));
                    } catch (ParseException e) {
                        e.printStackTrace();
                    }
                }
            });
            thread.start();//(4)启动线程
        }
    }
}
~~~

* 代码（1）创建了SimpleDateFormat的一个实例，
* 代码（2）创建10个线程，每个线程都公用同一个sdf对象对文本日期进行解析，多运行几次就会抛出java.lang.NumberFormatException异常，加大线程的个数有利于该问题复现

## 问题分析
为了便于分析首先奉上SimpleDateFormat的类图结构：



可知每个SimpleDateFormat实例里面有一个Calendar对象，从后面会知道其实SimpleDateFormat之所以是线程不安全的就是因为Calendar是线程不安全的，后者之所以是线程不安全的是因为其中存放日期数据的变量都是线程不安全的，比如里面的fields，time等。

下面从代码层面看下parse方法做了什么事情：
~~~
 public Date parse(String text, ParsePosition pos)
    {
       
        //(1)解析日期字符串放入CalendarBuilder的实例calb中
        .....

        Date parsedDate;
        try {//（2）使用calb中解析好的日期数据设置calendar
            parsedDate = calb.establish(calendar).getTime();
            ...
        }
       
        catch (IllegalArgumentException e) {
           ...
            return null;
        }

        return parsedDate;
    }
~~~

~~~
Calendar establish(Calendar cal) {
   ...
   //（3）重置日期对象cal的属性值
   cal.clear();
   //(4) 使用calb中中属性设置cal
   ...
   //(5)返回设置好的cal对象
   return cal;
}
~~~

* 代码（1）主要的作用是解析字符串日期并把解析好的数据放入了 CalendarBuilder的实例calb中，CalendarBuilder是一个建造者模式，用来存放后面需要的数据。
* 代码（3）重置Calendar对象里面的属性值，如下代码：
~~~
 public final void clear()
   {
       for (int i = 0; i < fields.length; ) {
           stamp[i] = fields[i] = 0; // UNSET == 0
           isSet[i++] = false;
       }
       areAllFieldsSet = areFieldsSet = false;
       isTimeSet = false;
   }
~~~

* 代码（4）使用calb中解析好的日期数据设置cal对象
* 代码（5） 返回设置好的cal对象
* 从上面步骤可知步骤（3）（4）（5）操作不是原子性操作，当多个线程调用parse
方法时候比如线程A执行了步骤（3）（4）也就是设置好了cal对象，在执行步骤（5）前线程B执行了步骤（3）清空了cal对象，由于多个线程使用的是一个cal对象，所以线程A执行步骤（5）返回的就可能是被线程B清空后的对象，当然也有可能线程B执行了步骤（4）被线程B修改后的cal对象。从而导致程序错误。

## 那么怎么解决那？

* 第一种方式：每次使用时候new一个SimpleDateFormat的实例，这样可以保证每个实例使用自己的Calendar实例,但是每次使用都需要new一个对象，并且使用后由于没有其它引用，就会需要被回收，开销会很大。
* 第二种方式：究其原因是因为多线程下步骤（3）（4）（5）三个步骤不是一个原子性操作，那么容易想到的是对其进行同步，让（3）（4）（5）成为原子操作，可以使用synchronized进行同步，具体如下：
~~~
public class TestSimpleDateFormat {
    // (1)创建单例实例
    static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static void main(String[] args) {
        // (2)创建多个线程，并启动
        for (int i = 0; i < 10; ++i) {
            Thread thread = new Thread(new Runnable() {
                public void run() {
                    try {// (3)使用单例日期实例解析文本
                        synchronized (sdf) {
                            System.out.println(sdf.parse("2017-12-13 15:17:27"));
                        }
                    } catch (ParseException e) {
                        e.printStackTrace();
                    }
                }
            });
            thread.start();// (4)启动线程
        }
    }
}
~~~
使用同步意味着多个线程要竞争锁，在高并发场景下会导致系统响应性能下降。

* 第三种方式：使用ThreadLocal，这样每个线程只需要使用一个SimpleDateFormat实例相比第一种方式大大节省了对象的创建销毁开销，并且不需要对多个线程直接进行同步，使用ThreadLocal方式代码如下：
~~~
public class TestSimpleDateFormat2 {
    // (1)创建threadlocal实例
    static ThreadLocal<DateFormat> safeSdf = new ThreadLocal<DateFormat>(){
        @Override 
        protected SimpleDateFormat initialValue(){
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };
    
    public static void main(String[] args) {
        // (2)创建多个线程，并启动
        for (int i = 0; i < 10; ++i) {
            Thread thread = new Thread(new Runnable() {
                public void run() {
                    try {// (3)使用单例日期实例解析文本
                            System.out.println(safeSdf.get().parse("2017-12-13 15:17:27"));
                    } catch (ParseException e) {
                        e.printStackTrace();
                    }
                }
            });
            thread.start();// (4)启动线程
        }
    }
}
~~~
> 代码（1）创建了一个线程安全的SimpleDateFormat实例，步骤（3）在使用的时候首先使用get()方法获取当前线程下SimpleDateFormat的实例，在第一次调用ThreadLocal的get（）方法适合会触发其initialValue方法用来创建当前线程所需要的SimpleDateFormat对象。