## Spring与线程安全

Spring作为一个IOC/DI容器，帮助我们管理了许许多多的“bean”。但其实，Spring并没有保证这些对象的线程安全，需要由开发者自己编写解决线程安全问题的代码。

Spring对每个bean提供了一个scope属性来表示该bean的作用域。它是bean的生命周期。例如，一个scope为singleton的bean，在第一次被注入时，会创建为一个单例对象，该对象会一直被复用到应用结束。

- singleton：默认的scope，每个scope为singleton的bean都会被定义为一个单例对象，该对象的生命周期是与Spring IOC容器一致的（但在第一次被注入时才会创建）。
- prototype：bean被定义为在每次注入时都会创建一个新的对象。
- request：bean被定义为在每个HTTP请求中创建一个单例对象，也就是说在单个请求中都会复用这一个单例对象。
- session：bean被定义为在一个session的生命周期内创建一个单例对象。
- application：bean被定义为在ServletContext的生命周期中复用一个单例对象。
- websocket：bean被定义为在websocket的生命周期中复用一个单例对象。

我们交由Spring管理的大多数对象其实都是一些无状态的对象，这种不会因为多线程而导致状态被破坏的对象很适合Spring的默认scope，每个单例的无状态对象都是线程安全的（也可以说只要是无状态的对象，不管单例多例都是线程安全的，不过单例毕竟节省了不断创建对象与GC的开销）。

无状态的对象即是自身没有状态的对象，自然也就不会因为多个线程的交替调度而破坏自身状态导致线程安全问题。无状态对象包括我们经常使用的DO、DTO、VO这些只作为数据的实体模型的贫血对象，还有Service、DAO和Controller，这些对象并没有自己的状态，它们只是用来执行某些操作的。例如，每个DAO提供的函数都只是对数据库的CRUD，而且每个数据库Connection都作为函数的局部变量（局部变量是在用户栈中的，而且用户栈本身就是线程私有的内存区域，所以不存在线程安全问题），用完即关（或交还给连接池）。

有人可能会认为，我使用request作用域不就可以避免每个请求之间的安全问题了吗？这是完全错误的，因为Controller默认是单例的，一个HTTP请求是会被多个线程执行的，这就又回到了线程的安全问题。当然，你也可以把Controller的scope改成prototype，实际上Struts2就是这么做的，但有一点要注意，Spring MVC对请求的拦截粒度是基于每个方法的，而Struts2是基于每个类的，所以把Controller设为多例将会频繁的创建与回收对象，严重影响到了性能。

通过阅读上文其实已经说的很清楚了，Spring根本就没有对bean的多线程安全问题做出任何保证与措施。对于每个bean的线程安全问题，根本原因是每个bean自身的设计。

**不要在bean中声明任何有状态的实例变量或类变量，如果必须如此，那么就使用ThreadLocal把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等这些实现线程同步的方法了。**


## Are Spring Beans Thread Safe?
 No.

Spring has different bean scopes (e.g. Prototype, Singleton, etc.) but all these scopes enforce is when the bean is created. For example a "prototype" scoped bean will be created each time this bean is "injected", whereas a "singleton" scoped bean will be created once and shared within the application context. There are other scopes but they just define a time span (e.g. a "scope") of when a new instance will be created.

The above has little, if anything to do with being thread safe, since if several threads have access to a bean (no matter the scope), it would only depend on the design of that bean to be or not to be "thread safe".

The reason I said "little, if anything" is because it might depend on the problem you are trying to solve. For example if you are concerned whether 2 or more HTTP requests may create a problem for the same bean, there is a "request" scope that will create a new instance of a bean for each HTTP request, hence you can "think" of a particular bean as being "safe" in the context of multiple HTTP requests. But it is still not truly thread safe by Spring since if several threads use this bean within the same HTTP request, it goes back to a bean design (your design of a bean backing class).

## How to Make/Design a Thread Safe "Object"?
There are several ways, probably too long to list here but here are a few examples:

- Design your beans **immutable**: for example have no setters and only use constructor arguments to create a bean. There are other ways, such as [Builder pattern](https://en.wikipedia.org/wiki/Builder_pattern), etc..

- Design your beans **stateless**: for example a bean that does something can be just a function (or several). This bean in most cases can and should be stateless, which means it does not have any state, it only does things with function arguments you provide each time (on each invocation)

- Design your beans [persistent](https://en.wikipedia.org/wiki/Persistent_data_structure): which is a special case of "immutable", but has some very nice properties. Usually is used in functional programming, where Spring (at least yet) not as useful as in imperative world, but I have used them with Scala/Spring projects.

- Design your beans with locks [last resort]: I would recommend against this unless you are working on a lower level library. The reason is we (humans) are not good thinking in terms of locks. Just the way we are raised and nurtured. Everything happens in parallel without us needing to "put that rain on pause, let me get an umbrella". Computers however are all about locks when you are talking "multiple things at the same time", hence there are some of us (exceptional people) who are doing their fair share and implementing libraries based on these locks. Most of other humans can just use these libraries and worry not about concurrency.


>引用
[are-spring-objects-thread-safe](https://stackoverflow.com/questions/15745140/are-spring-objects-thread-safe)
[spring-singleton-request-session-beans-and-thread-safety/](https://tarunsapra.wordpress.com/2011/08/21/spring-singleton-request-session-beans-and-thread-safety/)