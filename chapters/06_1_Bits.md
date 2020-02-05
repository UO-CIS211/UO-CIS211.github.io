---
layout: page
title:  Bit Twiddling
---

As you have undoubtably heard, the data and instructions
  manipulated by a modern digital computer are all in the
  form of binary numbers.  Often we think of them in 
  other ways:  As characters, as strings, as (decimal) integers, as 
  (decimal) floating point numbers, as whatever we choose 
  to represent.  Programming languages like Python help us
  mostly ignore that they are all really just binary numbers, 
  and usually that is a good thing, but sometimesit is useful 
   to peek under the covers and deal with the 
  binary representation more directly. 
  
In this chapter we will manipulate binary numbers that 
may have multiple _fields_ that may be packed together 
into a single integer _word_.  We will look at how to 
_pack_ small binary numbers into _fields_ of a larger 
binary number, and how to extract the small integers 
from the packed representation.  Python provides several 
_bitwise_ operations to make this relatively simple. 

## It's already binary

What's already binary?  Everything.  Everything in the computer is already 
binary. We will _not_ convert anything to binary.  If it is in computer 
memory, it is already binary, because that is the _only_ kind of information 
that modern computer memory can hold.  No exceptions. 

You may be under the impression that you have converted integers to binary 
form in the past.  In fact, Python has a built-in function called 
`bin`, and you may be under the impression that `bin` converts a number to 
binary: 

``` 
>>> x = 42
>>> bin(x)
'0b101010'
```

You have been tricked!  The `bin` function does not convert the integer to binary. 
Let's see what the type of the result for the `bin` function really is: 

``` 
>>> y = bin(x)
>>> type(y)
<class 'str'>
```

The `bin` function produces a string.  The string is a representation of how 
we might write the binary number ... but the number was already binary. 

But isn't that string also represented in binary?   OK, you've got me there ... 
all strings are also represented in binary, so in that sense `"0b101010"` is 
binary, bit no more so than `"Hello World"`. 

### So when and how was it converted to binary? 

When any kind of data, including your program source code, is read by the computer, 
it is converted to binary as part of the reading process.  When you strike the 'k' 
key on the keyboard, a binary code for the letter 'k' is transmitted to the computer. 
When you type the keys '4' and '2',  you are producing the binary codes for the 
character '4' and the character '2'.  If you typed `x = 42` in your Python program, 
the Python compiler converted those character codes to the internal representation 
0b101010.   

### But when I print it, I see '42'

When you print x, the internal representation 0b101010 is converted into the 
character codes for '4' and '2'.  At no time was the decimal integer 42 
ever present.  It's a pleasant illusion brought to you by Python.  
 
 
## The binary representation of an integer

Although _everything_ is represented in binary, in this chapter we will deal only with 
integers, because their representation in binary is simplest.  

When we write a decimal number on paper, we use a place value system:  The 
digits '42' mean $$4\times{10^1} + 2\times{10^0}$$, and the digits '203' mean 
$$2\times{10^2} + 0\times{10^1} + 3\times{10^0}$$. 

