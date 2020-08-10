---
layout: page
mathjax: true
title:  How Python Works
---

## Or: Seven things they never told you about Python.  Number 5 will blow your mind.

You write programs in Python to run with the Python interpreter, 
but the Python interpreter itself is a computer program. 
Let's look inside. 

## Use the source, Luke

Although there are alternative implementations of Python, when 
someone mentioned "the Python interpreter", they usually mean 
CPython.  That's the version of Python that you can download and 
install from `python.org`.  CPython 2.x and CPython 3.x are both 
versions of CPython. 

CPython is an open source project.  It is hosted on GitHub, 
like our projects, but it is somewhat larger.  You can 
find it at [`https://github.com/python/cpython`](https://github.com/python/cpython). 
Perhaps one day you will contribute a new feature to CPython, or fix a bug. 

As a relatively new developer, it is likely a little daunting to 
dive into a codebase of this size and complexity, but there is 
documentation to help you when you are ready. 
An overall guide for developers who want to contribute 
to the Python project can be found at 
[`https://devguide.python.org/`](https://devguide.python.org/). 
A more general guide for understanding Python's internals, even if you do not 
want to contribute to Python's development, can be found 
at [`https://devguide.python.org/exploring/`](https://devguide.python.org/exploring/). 

I have not yet watched Philip Guo's videos on CPython internals.  I expect 
them to be very good ... Philip is the creator of the wonderful 
[PythonTutor](http://pythontutor.com/).  You can find them at 
[`http://pgbovine.net/cpython-internals.htm`](http://pgbovine.net/cpython-internals.htm).

I expect these guides and the source code itself will be more accessible to 
you when you are a little farther along in computer science.  In the remainder 
of this document, I'll try to describe some key aspects of how CPython works, 
but in less detail and with fewer assumptions about your background. 

## Translation and Execution 

CPython is an interpreter. That means it reads Python source code and then 
executes instructions that "interpret" the program.  In CIS 211 you  
write an interpreter for a calculator, and in the final project you 
write a compiler for a simple language to run on the Duck Machine.  
CPython is more complex, but not essentially different. 

The CPython interpreter works in two phases.  In the first phase, CPython 
translates Python source code into an intermediate, lower-level representation. 
The lower level representation is designed to be efficient for execution, 
and is not readily readable by humans. It is a sequence of instructions for 
a Python _virtual machine_, which is a program simulating a special purpose 
Python computer. 

The Python virtual machine, or byte code interpreter, is a simulated 
computer like our Duck Machine.  Unlike our Duck Machine, though, it 
is not designed to be simulate a real hardware computer.  It is instead 
an imaginary machine specialized for executing Python programs.  It doesn't
have registers like the Duck Machine, but it has several data structures 
for managing Python program execution.  One of the key data structures 
is a _stack_.  Virtual machines like the one in CPython are called 
_stack machines_ because their _stack_ is used much the way 
registers are used in a physical computer. 

Each of the operations of the Python 
virtual machine is more complex than 
individual operations on the Duck Machine. 
Since the Python virtual machine is a stack 
machine, the byte code instructures are arranged 
in postfix order, much like the input of our calculator. 
For example, this small Python function: 

```python
def simple(a: int, b: int) -> int:
    x = 11
    c = a + b * x
    return c
```

is translated into the following byte code instructions: 

``` 
LOAD_CONST  1   # Push constant 11, which is at location 1
STORE_FAST  2   # Pop and store in x, at location 2
LOAD_FAST   0   # Push first argument a, found at address 0
LOAD_FAST   1   # Push second argument b, found at address 1
LOAD_FAST   2   # Push value of x 
BINARY_MULTIPLY # Pops values of x and b, pushes x*b
BINARY_ADD      # Pops values x*b and a,  pushes a + x*b
STORE_FAST  3   # Pop sum and store in variable c
LOAD_FAST   3   # Push value of c, found at address 3
RETURN_VALUE    # Pop value and return it
```

This is not very clever code!  Note how it pops a value to store
in variable `c`, then immediately pushes that value back on the 
stack before returning it.  When you write a code generator, 
do not despair over the quality of code you produce.  Code generators
written by experts are pretty bad too.  (Language translators often 
have an optional _optimization_ phase that improves the generated 
code somewhat.)

What does the `BINARY_ADD` byte code instruction do?  You already 
know:  It calls the `__add__` method on its operand.  This is a 
very complex operation compared to the simple operations performed
by each instruction in a physical CPU. 

## Python translation compared to Java, C, etc

Many language implementations use a two-step process 
much like Python.  If you have programmed in Java, 
the first part (translation to byte code) was what the 
`javac` program did, and the second part (executing 
the byte code) was what the `java` program did.  The 
Java virtual machine differs from the Python virtual 
machine in several regards, but it too is a stack machine. 

Some language implementations take a different approach, 
translating the source language (like C or C++) to 
the actual machine code of a physical machine. This is 
called "compiling" or "compiling to native code". 
The CPython interpreter itself is a C program that has 
been compiled to native code, which might be the machine 
code for an Intel x86 processor or the machine code for 
an ARM CPU (for example, if you run Python on a 
Raspberry Pi). 

In principle the same language can be translated in different 
manners.  There is nothing to prevent building an interpreter 
for C (and it has been done), and it is possible in principle 
to build a native code compiler for Python (but it's not easy). 
In practice, some language designs lend themselves to native 
code compilation, and some make it more difficult.  Python was 
designed to be an interpreted language. 

Further reading:  There is a very nice, basic description of 
the Python virtual stack machine at 
[https://www.ics.uci.edu/~brgallar/week9_3.html](https://www.ics.uci.edu/~brgallar/week9_3.html). 


## Key Data Types in CPython

The built-in object types in Python include integers, 
strings, lists, and dicts.  There is a built-in class 
for each of these.  The built-in classes are almost like 
classes you define in Python, but they are actually written 
in C. 

### Class int

Although memory hardware is not really "typed", CPUs have 
operations like ADD that treat values as integers.
Most programming languages have an integer type that 
is essentially just a 32-bit or 64-bit integer exactly 
as supported by the hardware.  Python's `int` class is 
different.  A Python `int` is actually an object wrapper for 
a variable length list of integers.  Python uses this 
representation so that it can deal with very large numbers, 
even larger than 64 bits which is typically the upper 
limit for integers in languages like Java and C. 
When you add two large integers, Python may actually execute 
a loop to add up corresponding parts!  

While the standard integer representation in all modern 
computers I am aware of uses 2s complement representation 
for negative numbers, Python uses sign and magnitude
representation.  This complicates every operation, although 
the interpreter uses some clever tricks to make arithmetic 
on small numbers reasonably efficient.  



Because the `int` class really is a class, it has methods 
like any other class.  When we add `int` objects, we call the 
`__add__` method just like we would if we were adding strings 
(whose `__add__` method concatenates strings).  The `__add__`
method for `int`, and all the other arithmetic methods for `int`, 
are implemented in C.  

## Class list



## References 

In addition to Python official documentation, 
I have relied partially on some of these: 

Brayan Rafael Gallardo, summary of class notes from UC Irvine 
ICS 33 (based on lectures and notes by Richard Pattis), 2016. 
[https://www.ics.uci.edu/~brgallar/index.html](https://www.ics.uci.edu/~brgallar/index.html)

Artem Golubin, "Python internals: Arbitrary-precision integer 
implementation", 2017. 
[https://rushter.com/blog/python-integer-implementation/](https://rushter.com/blog/python-integer-implementation/)


