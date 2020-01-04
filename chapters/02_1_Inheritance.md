---
layout: page
title: Inheritance
---

## Composing objects: Inheritance

Sometimes we need a specialized version of 
a class, or multiple variations on a class. 
Rather than modifying the existing class, 
we can create a new class that *inherits* 
from the existing class. 

Suppose, for example, that we wanted a 
class `Square` that is like a rectangle but 
can be created from a single point and a size. 
We can specify that a `Square` is a kind of 
`Rect`:

```python
class Square(Rect):
    def __init__(self, anchor: Point, size: Number):
        self.min_pt = anchor
        self.max_pt = self.min_pt + Point(size, size)
        self.size = size
``` 

`Square` objects have the `min_pt` and `max_pt` 
*instance variables* of `Rect`, plus a new 
instance variable `size`.  
All the methods of `Rect` are also available for 
`Square`, including the *magic* methods like `__str__`.  

```python
p1 = Point(3, 5)
sq = Square(p1, 5)
print(sq)
s2 = sq.translate(Point(2,2))
print(s2)
```

Running this code gives us

```
Rect((3, 5), (8, 10))
Rect((5, 7), (10, 12))
```

Maybe we don't want this.  Maybe we want 
`Square` to have it's own `__str__` method. 

```python
    def __str__(self) -> str:
        return f"Square({str(self.min_pt)}, {self.size})"
```

The new `__str__` method of `Square` *overrides* the 
`__str__` method of the `Rect` Now the same code as above 

```python
p1 = Point(3, 5)
sq = Square(p1, 5)
print(sq)
s2 = sq.translate(Point(2,2))
print(s2)
```

prints

```
Square((3, 5), 5)
Rect((5, 7), (10, 12))
```

The first `print` command uses the new 
`__str__` method. 
The `translate` method is still inherited from 
`Rect`, and still returns a `Rect` object rather 
than a `Square`, so the second `print` command 
is printing that `Rect` object. 

### Inheriting from built-ins 

All the built-in types of Python, like `str`
and `dict` and even `int`, are classes.  Just 
as we were able to extend or *inherit from* 
the class `Rect`, it is occasionally useful to 
inherit from one of the built-in classes.  