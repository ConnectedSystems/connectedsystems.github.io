---
title: Learning Julia
---

If you haven't heard of Julia, it is a new programming language that advertises high-performance (e.g. close to C or Fortran) while having a dynamic feel (like R or Python). The initial v1.0 was released quite recently (Aug 2018) and it is now up to v1.3 (released Nov 2019). One of the long-awaited features included in the latest release is automatic thread spawning (although it is marked as experimental).

Syntactically it looks vaguely like a mix of Matlab and Python. The biggest draw card of Julia is definitely the promised performance gain in comparison to Python or R.

I've been picking up Julialang on and off for the past two years or so now, but never really got stuck into it. In recent weeks I've decided to take on two relatively big side projects. The first is to move the agricultural management model I developed as part of my PhD to Python 3, which I've named Agtor. The second was to port Agtor across to Julia. The reasoning for this is three-fold: 

1) For various reasons Agtor is outgrowing the performance that Python can offer.
2) A clean Python implementation is still useful for demonstration and prototyping purposes (hence the port to Python 3 - not to mention Python 2 is end-of-life on 1 Jan 2020!)
3) I wanted to dive in and learn Julia with a non-trivial project

Some aspects of point 1 could be addressed by leveraging numba or Cython. From my admittedly brief experience with numba, its efficacy seems to be restricted to the computationally intensive - places where a lot of mathematical calculations occur. This is not really Agtor's situation.

Cython was used in the original project from which Agtor arose and as simple as it was to incorporate, and as great as the performance gains were, it could never replace all the Python code, and nor should it.

As with learning any new language there's a lot to take in. I'm writing down some lessons learnt here, comparing against how things might be done in Python.

No "classical" class structure
----------------------------------

To set the scene lets briefly go over object definition in Python. To create an object you would do something like:

```python
class Cup(object):
    def __init__(self, volume=0):
        # volume in ml
        self.volume = volume
```

Which defines a `Cup` object and a constructor through which you can define its intial state (defaulting to empty: `volume = 0`). You could use it like so:

```python
my_cup = Cup(200)
```

Which would instantiate a `Cup` filled with 200 ml of unknown liquid.

Lets extend the Cup with attributes and represent behaviour through methods. I, for some reason, also want some properties (in this case, dynamic setters and getters).

```python
class Cup(object):
    def __init__(self, volume=0, capacity=250):
        self.volume = volume  # volume in ml
        self.capacity = capacity  # max capacity

    def drink(self, amount):
        vol = self.volume
        self.volume = max(vol - amount, 0)

    def fill(self, amount):
        vol = self.volume
        cap = self.capacity
        self.volume = min(vol + amount, cap)
    
    @property
    def volume_in_L(self):
        """Get volume in litres"""
        return self.volume / 1000.0
    
    @volume_in_L.setter
    def volume_in_L(self, volume):
        """Set the cup to exact volume using litre values"""
        vol_ml = volume * 1000.0
        self.volume = min(vol_ml, self.capacity)
```

One more thing - I'd like to create other kinds of liquid vessels. Say, a mug:

```python
class Mug(Cup):
    def __init__(self, volume, capacity):
        super().__init__(volume, capacity)
```

The above is of course class inheritance - the Mug *inherits* all the attributes, properties and methods as defined for the `Cup`. 

Now, how do we do this in Julia, given that

1. Julia does not have classes, it has structs.

2. Structs do not support methods.

3. Julia does not support inheritance.

...to be continued...
