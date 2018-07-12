
## Overview

The Adapter [2] design pattern is one of the twenty-three well-known GoF design patterns that describe how to solve recurring design problems to design flexible and reusable object-oriented software, that is, objects that are easier to implement, change, test, and reuse.

The Adapter design pattern solves problems like: [3]

- How can a class be reused that does not have an interface that a client requires?
- How can classes that have incompatible interfaces work together?
- How can an alternative interface be provided for a class?
Often an (already existing) class can't be reused only because its interface doesn't conform to the interface clients require.

The Adapter design pattern describes how to solve such problems:

- Define a separate Adapter class that converts the (incompatible) interface of a class (Adaptee) into another interface (Target) clients require.
- Work through an Adapter to work with (reuse) classes that do not have the required interface.
The key idea in this pattern is to work through a separate Adapter that adapts the interface of an (already existing) class without changing it.
Clients don't know whether they work with a Target class directly or through an Adapter with a class that does not have the Target interface.
See also the UML class diagram below.

## Definition
An adapter allows two incompatible interfaces to work together. This is the real-world definition for an adapter. Interfaces may be incompatible, but the inner functionality should suit the need. The Adapter design pattern allows otherwise incompatible classes to work together by converting the interface of one class into an interface expected by the clients.

~~~java
public class AdapteeToClientAdapter implements Adapter {

    private final Adaptee instance;

    public AdapteeToClientAdapter(final Adaptee instance) {
         this.instance = instance;
    }

    @Override
    public void clientMethod() {
       // call Adaptee's method(s) to implement Client's clientMethod
    }

}
~~~