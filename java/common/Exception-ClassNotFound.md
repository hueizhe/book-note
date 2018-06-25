## [ClassNotFoundException vs NoClassDefFoundError](http://www.baeldung.com/java-classnotfoundexception-and-noclassdeffounderror)

### 1. Introduction

Both ClassNotFoundException and NoClassDefFoundError occur when the JVM can not find a requested class on the classpath. Although they look familiar, there are some core differences between these two.

In this tutorial, we’ll discuss some of the reasons for their occurrences and their solutions.

### 2. ClassNotFoundException

ClassNotFoundException is a checked exception which occurs when an application tries to load a class through its fully-qualified name and can not find its definition on the classpath.

This occurs mainly when trying to load classes using Class.forName(), ClassLoader.loadClass() or ClassLoader.findSystemClass(). Therefore, we need to be extra careful of java.lang.ClassNotFoundException while working with reflection.

For example, let’s try to load the JDBC driver class without adding necessary dependencies which will get us ClassNotFoundException:
~~~java
@Test(expected = ClassNotFoundException.class)
public void givenNoDrivers_whenLoadDriverClass_thenClassNotFoundException() 
  throws ClassNotFoundException {
      Class.forName("oracle.jdbc.driver.OracleDriver");
}
~~~

### 3. NoClassDefFoundError
NoClassDefFoundError is a fatal error. It occurs when JVM can not find the definition of the class while trying to:

- Instantiate a class by using the new keyword
- Load a class with a method call

The error occurs when a compiler could successfully compile the class, but Java runtime could not locate the class file. It usually happens when there is an exception while executing a static block or initializing static fields of the class, so class initialization fails.

Let’s consider a scenario which is one simple way to reproduce the issue. ClassWithInitErrors initialization throws an exception. So, when we try to create an object of ClassWithInitErrors, it throws ExceptionInInitializerError.

If we try to load the same class again, we get the NoClassDefFoundError:

~~~java
public class ClassWithInitErrors {
    static int data = 1 / 0;
}
~~~

~~~java
public class NoClassDefFoundErrorExample {
    public ClassWithInitErrors getClassWithInitErrors() {
        ClassWithInitErrors test;
        try {
            test = new ClassWithInitErrors();
        } catch (Throwable t) {
            System.out.println(t);
        }
        test = new ClassWithInitErrors();
        return test;
    }
}
~~~

Let us write a test case for this scenario:
~~~java
@Test(expected = NoClassDefFoundError.class)
public void givenInitErrorInClass_whenloadClass_thenNoClassDefFoundError() {
  
    NoClassDefFoundErrorExample sample
     = new NoClassDefFoundErrorExample();
    sample.getClassWithInitErrors();
}
~~~

### 4. Resolution

Sometimes, it can be quite time-consuming to diagnose and fix these two problems. The main reason for both problems is the unavailability of the class file (in the classpath) at runtime.

Let’s take a look at few approaches we can consider when dealing with either of these:

1. We need to make sure whether class or jar containing that class is available in the classpath. If not, we need to add it
2. If it’s available on application’s classpath then most probably classpath is getting overridden. To fix that we need to find the exact classpath used by our application
3. Also, if an application is using multiple class loaders, classes loaded by one classloader may not be available by other class loaders. To troubleshoot it well, it’s essential to know how classloaders work in Java

### 5. Summary

While both of these exceptions are related to classpath and Java runtime unable to find a class at run time, it’s important to note their differences.

Java runtime throws ClassNotFoundException while trying to load a class at runtime only and the name was provided during runtime. In the case of NoClassDefFoundError, the class was present at compile time, but Java runtime could not find it in Java classpath during runtime.

As always, the complete code for all examples can be found over on [GitHub](https://github.com/eugenp/tutorials/tree/master/core-java).

