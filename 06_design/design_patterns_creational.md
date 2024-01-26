# Design Patterns - Creational

Creational patterns target the process of object creation. 

## Singleton

:::{admonition} Singleton
:class: note  
Singleton is a creational design pattern that lets you ensure that a class has only one instance, 
while providing a global access point to this instance.
:::


In its classic form it looks like this.

```java
public class ClassicSingleton {
    //PART ONE: THE SINGLE INSTANCE
    private static ClassicSingleton instance;

    //PART TWO: THE PRIVATE CONSTRUCTOR
    private ClassicSingleton() {
    }

    //PART THREE: THE PUBLIC STATIC GETTER
    public static ClassicSingleton getInstance() {
        if (instance == null) {
            //LAZY INITIALIZATION!
            instance = new ClassicSingleton();
        }
        return instance;
    }
}
```

However, this solution is not thread safe. Therefore, other implementations of this pattern have evolved.
Here is the solution using `synchronized` and `volatile`:

```java
public class ThreadSafeSingleton {
    //volatile -- keep in main memory, not in thread-local cache
    private static volatile ThreadSafeSingleton instance = null;

    private ThreadSafeSingleton() {
    }

    public static ThreadSafeSingleton getInstance() {
        if (instance == null) {
            //synchronized -- ensures that only one thread can enter this block at a time
            synchronized (ThreadSafeSingleton.class) {
                if (instance == null) {
                    instance = new ThreadSafeSingleton();
                }
            }
        }
        return instance;
    }
}
```

:::{admonition} `synchronized`
:class: note  
Java synchronized blocks can be used to avoid race conditions (a system attempts to perform two 
or more operations at the same time, but the operations must be done in the proper sequence to be done correctly).   
A synchronized block in Java is synchronized on some object.  
All synchronized blocks synchronized on the same object can only have one thread executing inside them at the same time.  
:::

:::{admonition} `volatile`
:class: note  
Volatile is used to indicate that a variable's value will be modified by different threads.  
Declaring a volatile Java variable means:  

* The value of this variable will never be cached thread-locally: all reads and writes will go straight to "main memory”.
* Access to the variable acts as though it is enclosed in a synchronized block, synchronized on itself.
:::

Other -arguably simpler- solutions make use of `enum` properties:

```java
public enum Singleton {
    INSTANCE;
}
```

However, any instance variables need to be specified as compile-time constants. This is often not appropriate or applicable.

GIYF for more in-depth discussion of the different solutions.

GGcdi54@v5ad3bH

## Factory (Method)

Factory pattern comes in several flavors, depending on the complexity of your model: 

- Factory Method
- Factory (class)
- Abstract Factory

Here, we'll only discuss factory method.

In Factory pattern, we create an object without exposing the creation logic to the client,
 and refer to newly created object using a common interface.  

More advanced (Abstract Factory):  
Define an interface for creating an object, but let subclasses decide which class to instantiate.  

The Factory method lets a class defer instantiation it uses to subclasses.


## Builder


