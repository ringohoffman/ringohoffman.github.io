---
title: "Python: A Strongly, Dynamically, Duck Typed, Interpreted Language"
subtitle: "What is Python anyway?"
date:    2023-08-17T19:00:00-08:00
lastmod: 2023-08-31T21:17:40-08:00
draft: false
author: "Matthew Hoffman"
authorLink: "../about"
description: "Disambiguating Python and its runtime and static type systems with definitions and examples."
license: ""
images: []

tags: ["Python", "Typing", "Programming Languages"]
categories: ["Python"]

featuredImage: ""
featuredImagePreview: ""

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: false

toc:
  enable: true
  auto: true
code:
  copy: true
  maxShownLines: 50
math:
  enable: false
  # ...
mapbox:
  # ...
share:
  enable: true
  # ...
comment:
  enable: true
  # ...
library:
  css:
    # someCSS = "some.css"
    # located in "assets/"
    # Or
    # someCSS = "https://cdn.example.com/some.css"
  js:
    # someJS = "some.js"
    # located in "assets/"
    # Or
    # someJS = "https://cdn.example.com/some.js"
seo:
  images: []
resources:
- name: pdf-file-:counter
  params:
    icon: pdf
  src: '**.pdf'
  # ...
---

## Background and Motivation

I recently had a conversation with a few of my colleagues where I had mentioned that Python is a strongly typed language. One of them didn't think that it was, and another explained that strong typing means that every object has a type, which is true about Python but is not the definition of strong typing. It surprised me to realize that even though my colleagues and I are experienced Python developers, none of us had a clear understanding of strong typing. This inspired me to write this post to help disambiguate Python and its runtime and static type systems.

## Compiled vs. Interpreted Languages

A language itself is neither compiled nor interpreted. The terms "interpreted language" or "compiled language" signify that the canonical implementation of that language is an interpreter or a compiler, respectively. While interpretation and compilation are the two main means by which programming languages are implemented, they are not mutually exclusive, as most interpreting systems also perform some translation work, just like compilers. [[1]](#1)

Compiled language implementations use a [compiler](https://en.wikipedia.org/wiki/Compiler) to translate a program's source code to the [instructions](https://en.wikipedia.org/wiki/Instruction_set_architecture#Instructions) of a target machine ([machine code](https://en.wikipedia.org/wiki/Machine_code)). For example, a `+` operation in source code could be compiled directly to the machine code `ADD` instruction. [[2]](#2)

Interpreted language implementations use an [interpreter](https://en.wikipedia.org/wiki/Interpreter_(computing)) to directly execute instructions without requiring them to have been compiled to machine code. An interpreter generally uses one of the following strategies for program execution:

1. Parse the source code and perform its behavior directly. (Early versions of Lisp and BASIC)
2. Translate source code into some efficient intermediate representation or object code and immediately execute that. (Python, Perl, MATLAB, and Ruby)
3. Explicitly execute stored precompiled bytecode made by a compiler and matched with the interpreter Virtual Machine. (UCSD Pascal)

Some implementations may also combine two and three, such as contemporary versions of Java. [[1]](#1)

Python has both compiled and interpreted implementations. [CPython](https://en.wikipedia.org/wiki/CPython), Python's [reference implementation](https://en.wikipedia.org/wiki/Reference_implementation), can be defined as both an interpreter and a compiler, as it compiles Python code into [bytecode](https://en.wikipedia.org/wiki/Bytecode) before interpreting it. [[3]](#3) [PyPy](https://en.wikipedia.org/wiki/PyPy), another Python implementation, is a just-in-time (JIT) compiler that compiles Python code into machine code at runtime.

## Dynamic vs. Static Typing

In statically typed languages, variables have declared or inferred types, and a variable's type cannot change. In dynamically typed languages like Python, values (runtime objects) have types, and variables are free to be reassigned to values of different types: [[4]](#4)

```python
a = 42
a = "hello world!"
```

## Strong vs. Weak Typing

There is no precise technical definition of what the "strength" of a type system means, [[5]](#5) but for dynamically typed languages, it is generally used to refer to how primitives and library functions respond to different types.

Python is considered strongly typed since most type changes require explicit conversions. A string containing only digits doesn't magically become a number, as in JavaScript (a weakly typed language):

```javascript
>>> "1" + 10
"110"
```

In Python, `+` works on two numbers or two strings, but not a number and a string. This was a design choice made when `+` was implemented for these classes and not a necessity following from Python's semantics. Observe that even though strongly typed, Python is completely fine with adding objects of type `int` and `float` and returns an object of type `float` (e.g., `int(42) + float(1)` returns `43.0`), and when you implement `+` for a custom type, you can implicitly convert anything to a number. [[4]](#4)

## Nominal vs. Structural Typing

Nominal typing is a static typing system that determines type compatibility and equivalence by explicit declarations and/or name matching. This means two variables are type-compatible if their declarations name the same type. [[6]](#6) In general, Python's static type system is nominally typed:

```python
def add_ints(a: int, b: int) -> int:
    return a + b

add_ints(1, 1)  # OK

add_ints(1.0, 1.0)  # static typing error: "float" is not a subtype of "int"
```

Abstract base classes (see [`abc`](https://docs.python.org/3/library/abc.html)) allow you to define interfaces via inheritance, i.e. nominally. A class is abstract (not instantiable) if it inherits from [`abc.ABC`](https://docs.python.org/3/library/abc.html#abc.ABC) or its [metaclass](https://docs.python.org/3/reference/datamodel.html#metaclasses) is [`abc.ABCMeta`](https://docs.python.org/3/library/abc.html#abc.ABCMeta) and it has at least one abstract method (a method decorated by [`abc.abstractmethod`](https://docs.python.org/3/library/abc.html#abc.abstractmethod)). All abstract methods must be implemented by a subclass in order for that subclass to be concrete (instantiable).

{{< admonition type=tip title="When to use Protocols vs. ABCs " open=true >}}
While not all `ABC` methods need to be abstract, if an `ABC` doesn't have any non-abstract methods, I recommend using [`typing.Protocol`](#Protocol) as a less restrictive alternative.
{{< /admonition >}}

[`abc.ABC.register`](https://docs.python.org/3/library/abc.html#abc.ABCMeta.register) allows you to register an abstract base class as a "virtual superclass" of an arbitrary type. This allows virtual subclasses to pass runtime type checks like `isinstance` and `issubclass`, but has no effect on static type compatibility:

```python
import abc

class MyTuple(abc.ABC):
    @abc.abstractmethod
    def something_special(self) -> None:
        ...

MyTuple.register(tuple)

assert issubclass(tuple, MyTuple)  # OK

def do_something(obj: MyTuple) -> None:
    ...

do_something((1, 2, 3))  # static typing error: "tuple" is incompatible with "MyABC"
```

Structural typing is a static typing system that determines type compatibility and equivalence by the type's actual structure or definition. [[7]](#7) `collections.abc.Iterable` is an example of a structural type in Python. It accepts any type that implements the `__iter__` method.

```python
from collections.abc import Iterable, Iterator

def add_numbers(numbers: Iterable[int]) -> int:
    return sum(n for n in numbers)

class MyIterable:
    def __iter__(self) -> Iterator[int]:
        return iter([0, 1, 2])

add_numbers([0, 1, 2])     # OK
add_numbers((0, 1, 2))     # OK
add_numbers({0, 1, 2})     # OK
add_numbers(range(3))      # OK
add_numbers(MyIterable())  # OK

add_numbers(1)      # static typing error: "__iter__" is not present
add_numbers("012")  # static typing error: "str" is not a subtype of "int"
```

[`collections.abc`](https://docs.python.org/3/library/collections.abc.html#collections-abstract-base-classes) provides a set of common abstract base classes that are useful for typing function parameters based on the operations performed on them, namely:

| operation      | magic method    |
|----------------|-----------------|
| `... in x`     | `__contains__`  |
| `for ... in x` | `__iter__`      |
| `next(x)`      | `__next__`      |
| `reversed(x)`  | `__reversed__`  |
| `len(x)`       | `__len__`       |
| `x(...)`       | `__call__`      |
| `x[...]`       | `__getitem__`   |
| `x[...] = ...` | `__setitem__`   |
| `del x[...]`   | `__delitem__`   |
| `hash(x)`      | `__hash__`      |

<a id="Protocol"></a>

Similar to `abc.ABC`, subclasses of `collections.abc` classes are nominally typed, which limits their usefulness for static typing.

[`typing.Protocol`](https://docs.python.org/3/library/typing.html#typing.Protocol) provides a way to define **custom** structural types in Python. Any class that defines the same attributes and methods as a `Protocol` is considered to be a static subtype of that `Protocol`:

```python
from typing import Protocol

class HasName(Protocol):
    name: str

class Person:
    def __init__(self, name: str) -> None:
        self.name = name

class Place:
    def __init__(self, name: str) -> None:
        self.name = name

def print_name(obj: HasName) -> None:
    print(obj.name)

print_name(Person("Matthew"))        # OK
print_name(Place("San Francisco"))   # OK
print_name("Austin")                 # static typing error: "name" is not present
```

[`typing.runtime_checkable`](https://docs.python.org/3/library/typing.html#typing.runtime_checkable) is a decorator for `Protocol` classes that allows you to perform runtime type checks against `Protocol`s (with one exception):

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class HasName(Protocol):
    name: str

class Person:
    def __init__(self, name: str) -> None:
        self.name = name

assert isinstance(Person("Matthew"), HasName)  # OK
assert issubclass(Person, HasName)             # TypeError: Protocols with non-method members don't support issubclass()
```

{{< admonition type=note title="runtime_checkable Protocols only check for structure at runtime, not signatures or types" open=true >}}
`isinstance` or `issubclass` checks against `runtime_checkable` `Protocol`s only check for the presence of the required methods or attributes, not their type signatures or types.
{{< /admonition >}}

## Duck Typing

Python's runtime type system is duck typed. Duck typing is similar to structural typing but differs in that:

1. Duck typing is a dynamic typing system.
2. Compatibility is determined by checking only the parts of a type's structure that are accessed at runtime.

It gets its name from the [duck test](https://en.wikipedia.org/wiki/Duck_test): [[8]](#8)

{{< admonition type=quote title="Duck test" open=true >}}
If it walks like a duck and it quacks like a duck, then it must be a duck.
{{< /admonition >}}

```python
class Duck:
    def walk(self):
        ...

    def quack(self):
        ...

    def fly(self):
        ...

duck = Duck()

class Person:
    def walk(self):
        ...

    def quack(self):
        ...

person = Person()

def do_duck_stuff(obj):
    obj.walk()
    obj.quack()

do_duck_stuff(duck)
do_duck_stuff(person)  # works, a "Person" can walk and quack like a "Duck"!

def go_flying(obj):
    obj.fly()

go_flying(duck)
go_flying(person)  # AttributeError: "Person" object has no attribute "fly"
```

`Protocol`s are a natural complement to duck typing since neither use inheritance to determine type compatibility. In our example above, we could define and use:

```python
from typing import Protocol

class CanWalkAndQuack(Protocol):
    def walk(self):
        ...

    def quack(self):
        ...

def do_duck_stuff(obj: CanWalkAndQuack) -> None:
    obj.walk()
    obj.quack()

do_duck_stuff(duck)    # OK
do_duck_stuff(person)  # OK

class CanFly(Protocol):
    def fly(self):
        ...

def go_flying(obj: CanFly) -> None:
    obj.fly()

go_flying(duck)    # OK
go_flying(person)  # static typing error: "fly" is not present
```

## Recap

> Is Python an interpreted language?

Python is considered an interpreted language because its reference implementation, CPython, compiles Python code into bytecode at runtime before interpreting it.

> Is Python a dynamically typed language?

Python is a dynamically typed language because it allows variables to be reassigned to values of different types.

> Is Python a strongly typed language?

Python is considered a strongly typed language because most type changes require explicit conversions.

> Is Python's static type system nominally or structurally typed?

Python's static type system is generally nominally typed, but `collections.abc` provides a collection of useful structural types, and `typing.Protocol` provides a way to define custom structural types.

> Is Python a duck typed language?

Python's runtime type system is duck typed because it determines type compatibility by checking only the parts of an object's structure that are accessed at runtime.

## Sources

<a id="1"></a>[1] [Wikipedia: Interpreter (computing)](https://en.wikipedia.org/wiki/Interpreter_(computing))

<a id="2"></a>[2] [StackOverflow: Compiled vs. Interpreted Languages - Answer by mikera](https://stackoverflow.com/a/3265602/7059681)

<a id="3"></a>[3] [Wikipedia: CPython](https://en.wikipedia.org/wiki/CPython)

<a id="4"></a>[4] [StackOverflow: Is Python strongly typed? - Answer by community wiki](https://stackoverflow.com/a/11328980/7059681)

<a id="5"></a>[5] [Wikipedia: Strong and weak typing](https://en.wikipedia.org/wiki/Strong_and_weak_typing)

<a id="6"></a>[6] [Wikipedia: Nominal type system](https://en.wikipedia.org/wiki/Nominal_type_system)

<a id="7"></a>[7] [Wikipedia: Structural type system](https://en.wikipedia.org/wiki/Structural_type_system)

<a id="8"></a>[8] [Wikipedia: Duck typing](https://en.wikipedia.org/wiki/Duck_typing)
