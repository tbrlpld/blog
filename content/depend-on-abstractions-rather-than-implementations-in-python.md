Title: How to Depend on Abstractions, Rather Than Implementations (in Python)
Date: 2020-05-15 15:00
Category: Software Engineering
Tags: Python, Clean Architecture, Dependency Inversion
Slug: depend-on-abstractions-rather-than-implementations-in-python
Authors: Tibor
Summary: According to Rober C. Martin, we should avoid dependencies on concrete implementations as much as possible. I was wondering how that could be achieved in Python.
Draft: False

I am currently reading [Clean Architecture by Robert C. Martin (Uncle Bob)](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure-ebook-dp-B075LRM681/dp/B075LRM681/ref=mt_kindle?_encoding=UTF8&me=&qid=).
In the chapter on dependency inversion, he writes:
> Don’t refer to volatile concrete classes. Refer to abstract interfaces instead. This rule applies in all languages, wether statically or dynamically typed.

Of course, this is not always possible.
For example, the `str` class in Python is a concrete implementation.
It would also be ridiculous trying to avoid using it directly.
But, depending on `str` is not an issue that the dependency inversion principle is really concerned with.
This is because the `str` class is very **stable.**

Not all classes and dependencies are as stable as the `str` class of the Python core, though.
We should especially avoid depending on our own classes that are under current development and constantly changing.

## How Can We Avoid Depending on Concrete Implementations?

To use an implementation, you need to know about it.
Unless you use a pattern like dependency injection (let somebody else tell you the implementation to use — but that is something for another post) you gonna need to import the concrete class.

To limit the dependency on concrete implementations, we need to create abstract interfaces (or abstract base classes in Python, see the [`abc` package of the standard library](https://pymotw.com/3/abc/)).
The concrete implementation inherit from the abstract base class and fill it with functionality (the concrete implementation).

Now, the code that is using the concrete implementation (the consumer) should only rely on the functionality that is defined by the abstract base class.

## How Can We Make Sure to Only Use Features of the Abstraction?

We could look only at the abstract class while working with the concrete one and only use the methods and attributes defined in the abstract base class.
Yea… that sounds fun. Or we can use a type checker like [`mypy`](http://mypy-lang.org/).

In our consumer we can define the instance of the concrete class as being of the abstract base class’ type.
If we do so, `mypy` will show a warning that we are using an attribute that is not defined by the type we said we were using.

## Let’s Implement This

First, we define some abstract class (in `abstract.py`).
```python
import abc


class AbstractBase(abc.ABC):
    """Abstract class."""

    @abc.abstractclassmethod
    def multiplystring(self, single_string: str, muliplier: int) -> str:
        """Create long string with multiple versions of given string."""

```

Next, we create an implementation of the abstract class (in `concrete.py`).
```python
import abstract


class ConcreteClass(abstract.AbstractBase):
    """Concrete implementation."""

    def multiplystring(self, single_string: str, multiplier: int) -> str:
        return single_string * multiplier

    def doublestring(self, single_string: str) -> str:
        return single_string * 2
```

Our `ConcreteClass` actually implements the functionality defined by the abstract base class, but also adds an another method.
The consumer of the concrete class should never use the additional methods.
If it does, then it depends on the concrete implementation.
And that should be avoided as much as possible.

Now, let’s create a consumer (in `consumer.py`) that uses the concrete implementation, but should only rely on what is defined in the abstract base class.
```python
import concrete


concrete_obj1 = concrete.ConcreteClass()
print(concrete_obj1.multiplystring("something", 3))
print(concrete_obj1.doublestring("something"))

```

No warnings so far. We are using an interface that we shouldn't and `mypy` is not warning us.

This is because we have not "told" `mypy` yet that we only want to use the "interface" (attributes and methods) defined by the abstract base class.

Let’s tell `mypy` that we only want to use the functionality of the abstract base class.
To do so, we add a type annotation to declare our variable to be of the abstract base class’ type.
```python
import abstract  # add this import
import concrete

...

 # define type as abstract base class
concrete_obj2: abstract.AbstractBase = concrete.ConcreteClass()

print(concrete_obj2.multiplystring("something", 3))
print(concrete_obj2.doublestring("something"))
```

Let’s run `mypy` on this file.
```sh
$ mypy consumer.py
consumer.py:28: error: "AbstractBase" has no attribute "doublestring"
Found 1 error in 1 file (checked 1 source file)
```

Nice. So `mypy` is now warning us, that we are trying to use functionality that is not defined by the type we said we wanted to use.

## How Is This Helpful?

While somebody is developing the `ConcreteClass`, they might consistently add and remove methods and attributes.
In the meantime, we are working on the consumer and might want to make use of some features that is already part of the concrete class.

If we don’t have an abstract interface in-between the consumer and the concrete implementation, we might start using functionality that is later removed from the concrete implementation.

On the other hand, if we do have an abstract base class, then we on the consumer side know which functions we can rely on using.
Also, the abstract base class tells the developers of the concrete implementation which attributes and methods should be included in the concrete implementation.
They can extend this functionality, but they can not remove any features (otherwise their class remains abstract and can not be instantiated).

One requirement here is, of course, that the interface is stable.
If somebody keeps adding and removing methods and attributes in the abstract base class, then nothing is won.
But, it is much easier to keep something abstract stable than it is to keep a concrete implementation stable.

## Can We Completely Remove the Reference to the Concrete Implementation?

No, not really. If we want to use something, we need to define its use at some point.
Meaning somewhere we need to import it and create the source dependency.

What you can do is remove the dependency from the consumer and let a different layer tell the consumer which concrete class to use.
This would be the [dependency injection pattern](https://en.wikipedia.org/wiki/Dependency_injection).
I will not go into this too deep here, because that is its own huge topic.

Just quick, the dependency injection pattern involves four roles:

* Client — wants to use a service
* Interface — defines how the client wants to use a service
* Service — the object being used by the client
* Injector — creates the service and tells the client to use it

In our example: the client is our consumer, the interface is the abstract base class and the service is the concrete class.

What we don’t have is the injector.

To enable the injection, we would need to change our consumer from a script into a function.
```python
import abstract

def consumer_func(obj: abstract.AbstractBaseClass) -> None:
    obj.multiplystring("something", 3)
```

Our consumer is now a function and declares that it expects an object of type `abstract.AbstractBaseClass` to be passed in.
Inside the function we make use of the methods of the abstract base class.
If we were to reference methods not defined in it, `mypy` would warn us again.

Now where is that object coming from?
At this point we would need an additional module that defines the injector.
Something like this (in `injector.py`):
```python

import consumer
import concrete

concrete_obj =  concrete.ConcreteClass()
consumer.consumer_func(concrete_obj)

```

Instead of running the consumer directly, we are running it through the injector.
The injector also tells the consumer which concrete implementation to use.

Why would we want to do this?

The advantages you get by using the dependency injection have mainly to do with flexibility and ease of testing.
You could now switch out the concrete implementation without having to change the consumer.
This is particularly handy during testing.
You could even test the consumer without having any concrete implementation, yet.
The [Wikipedia article on dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) goes into more detail.

I only wanted to mention it here, because it seemed like a natural next step.

***

Hope you found this article helpful.
If I missed something or got this all wrong, or all right, [let me know](https://twitter.com/tbrlpld).