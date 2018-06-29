## Abstract factory pattern

The abstract factory pattern provides a way to encapsulate a group of individual factories that have a common theme without specifying their concrete classes.[1] In normal usage, the client software creates a concrete implementation of the abstract factory and then uses the generic interface of the factory to create the concrete objects that are part of the theme. The client doesn't know (or care) which concrete objects it gets from each of these internal factories, since it uses only the generic interfaces of their products.[1] This pattern separates the details of implementation of a set of objects from their general usage and relies on object composition, as object creation is implemented in methods exposed in the factory interface.[2]

An example of this would be an abstract factory class DocumentCreator that provides interfaces to create a number of products (e.g. createLetter() and createResume()). The system would have any number of derived concrete versions of the DocumentCreator class like FancyDocumentCreator or ModernDocumentCreator, each with a different implementation of createLetter() and createResume() that would create a corresponding object like FancyLetter or ModernResume. Each of these products is derived from a simple abstract class like Letter or Resume of which the client is aware. The client code would get an appropriate instance of the DocumentCreator and call its factory methods. Each of the resulting objects would be created from the same DocumentCreator implementation and would share a common theme (they would all be fancy or modern objects). The client would only need to know how to handle the abstract Letter or Resume class, not the specific version that it got from the concrete factory.

A factory is the location of a concrete class in the code at which objects are constructed. The intent in employing the pattern is to insulate the creation of objects from their usage and to create families of related objects without having to depend on their concrete classes.[2] This allows for new derived types to be introduced with no change to the code that uses the base class.

Use of this pattern makes it possible to interchange concrete implementations without changing the code that uses them, even at runtime. However, employment of this pattern, as with similar design patterns, may result in unnecessary complexity and extra work in the initial writing of code. Additionally, higher levels of separation and abstraction can result in systems that are more difficult to debug and maintain.

## Overview

The Abstract Factory [3] design pattern is one of the twenty-three well-known GoF design patterns that describe how to solve recurring design problems to design flexible and reusable object-oriented software, that is, objects that are easier to implement, change, test, and reuse.

The Abstract Factory design pattern solves problems like: [4]

How can an application be independent of how its objects are created?
How can a class be independent of how the objects it requires are created?
How can families of related or dependent objects be created?
Creating objects directly within the class that requires the objects is inflexible because it commits the class to particular objects and makes it impossible to change the instantiation later independently from (without having to change) the class. It stops the class from being reusable if other objects are required, and it makes the class hard to test because real objects can't be replaced with mock objects.

The Abstract Factory design pattern describes how to solve such problems:

Encapsulate object creation in a separate (factory) object. That is, define an interface (AbstractFactory) for creating objects, and implement the interface.
A class delegates object creation to a factory object instead of creating objects directly.
This makes a class independent of how its objects are created (which concrete classes are instantiated). A class can be configured with a factory object, which it uses to create objects, and even more, the factory object can be exchanged at run-time.
See also the UML class and sequence diagram below.

## Definition

Provide an interface for creating families of related or dependent objects without specifying their concrete classes

## Java example
~~~java
public interface IButton {
	void paint();
}

public interface IGUIFactory {
	public IButton createButton();
}

public class WinFactory implements IGUIFactory {
	@Override
	public IButton createButton() {
		return new WinButton();
	}
}

public class OSXFactory implements IGUIFactory {
	@Override
	public IButton createButton() {
		return new OSXButton();
	}
}

public class WinButton implements IButton {
	@Override
	public void paint() {
		System.out.println("WinButton");
	}
}

public class OSXButton implements IButton {
	@Override
	public void paint() {
		System.out.println("OSXButton");
	}
}

public class Main {

	public static void main(final String[] arguments) throws Exception {
		IGUIFactory factory = null;
		
		final String appearance = randomAppearance();	// Current operating system

		if (appearance.equals("OSX")) {
			factory = new OSXFactory();
		} else if(appearance.equals("Windows")) {
			factory = new WinFactory();
		} else {
			throw new Exception("No such operating system");
		}
		
		final IButton button = factory.createButton();

		button.paint();
	}
	
	/**
	 * This is just for the sake of testing this program, and doesn't have to do
	 * with Abstract Factory pattern.
	 * @return
	 */
	public static String randomAppearance() {
		final String[] appearanceArray = new String[3];

		appearanceArray[0] = "OSX";
		appearanceArray[1] = "Windows";
		appearanceArray[2] = "error";
		final java.util.Random random = new java.util.Random();

		final int randomNumber = random.nextInt(3);

		return appearanceArray[randomNumber];
	}
}
~~~


## Python example
~~~python
from __future__ import print_function

from abc import ABCMeta, abstractmethod

class Button:
    __metaclass__ = ABCMeta

    @abstractmethod
    def paint(self):
        pass

class LinuxButton(Button):
    def paint(self):
        return "Render a button in a Linux style"

class WindowsButton(Button):
    def paint(self):
        return "Render a button in a Windows style"

class MacOSButton(Button):
    def paint(self):
        return "Render a button in a MacOS style"

class GUIFactory:
    __metaclass__ = ABCMeta

    @abstractmethod
    def create_button(self):
        pass

class LinuxFactory(GUIFactory):
    def create_button(self):
        return LinuxButton()

class WindowsFactory(GUIFactory):
    def create_button(self):
        return WindowsButton()

class MacOSFactory(GUIFactory):
    def create_button(self):
        return MacOSButton()

appearance = "linux"

if appearance == "linux":
    factory = LinuxFactory()
elif appearance == "osx":
    factory = MacOSFactory()
elif appearance == "win":
    factory = WindowsFactory()
else:
    raise NotImplementedError(
        "Not implemented for your platform: {}".format(appearance)
    )

if factory:
    button = factory.create_button()
    result = button.paint()
    print(result)
~~~
Alternative implementation using the classes themselves as factories:

~~~python
from __future__ import print_function

from abc import ABCMeta, abstractmethod

class Button:
    __metaclass__ = ABCMeta

    @abstractmethod
    def paint(self):
        pass

class LinuxButton(Button):
    def paint(self):
        return "Render a button in a Linux style"

class WindowsButton(Button):
    def paint(self):
        return "Render a button in a Windows style"

class MacOSButton(Button):
    def paint(self):
        return "Render a button in a MacOS style"

appearance = "linux"

if appearance == "linux":
    factory = LinuxButton
elif appearance == "osx":
    factory = MacOSButton
elif appearance == "win":
    factory = WindowsButton
else:
    raise NotImplementedError(
        "Not implemented for your platform: {}".format(appearance)
    )

if factory:
    button = factory()
    result = button.paint()
    print(result)

~~~