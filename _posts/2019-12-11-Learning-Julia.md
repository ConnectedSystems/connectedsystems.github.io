---
title: Learning Julia
---

If you haven't heard of Julia, it is a new programming language that advertises high performance (e.g. close to C or Fortran) while having a dynamic feel like R or Python. The promised performance gains is certainly the biggest draw card that Julia has compared to Python or R. The initial v1.0 was released quite recently (Aug 2018) and it is now up to v1.3 (released Nov 2019). One of the long-awaited features included in the latest release is automatic thread spawning (although it is marked as experimental). Syntactically it looks vaguely like a mix of Matlab and Python.

I've been picking up Julialang on and off for the past two years or so now, but never really got stuck into it. In recent weeks I've decided to take on two relatively big side projects. The first is to move the agricultural management model I developed as part of my PhD to Python 3, which I've named Agtor. The second was to port Agtor across to Julia. The reasoning for this is three-fold: 

1) For various reasons Agtor is outgrowing the performance that Python can offer.

2) A clean Python implementation is still useful for demonstration and prototyping purposes (hence the port to Python 3 - not to mention Python 2 is end-of-life on 1 Jan 2020!)

3) I wanted to dive in and learn Julia with a non-trivial project

Some aspects of point 1 could be addressed by leveraging numba or Cython. From my admittedly brief experience with numba, its efficacy seems to be restricted to the computationally intensive - places where a lot of mathematical calculations occur. This is not really Agtor's situation, as most of the runtime is attributable to calling C/Fortran libraries. A lot of the time is spent on loops and function/method calls.

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

Now, how do we do this in Julia, given that it doesn't support C++ style Object-Orientation.

1. Julia does not have classes (it has structs).

2. Structs do not support methods.

3. Julia does not support inheritance in the vein of C++.

Well, here's a full code dump:

```julia
# The base type...
mutable struct Cup
    volume
    capacity
end

# functions tied to a type and its subtypes
function drink(c::Cup, amount)
    c.volume = max(c.volume - amount, 0)
end

function fill(c::Cup, amount)
    vol = self.volume
    cap = self.capacity
    c.volume = min(vol + amount, cap)
end

# to replicate class properties...
function Base.getproperty(c::Cup, v::Symbol)
    if v == :volume_in_L
        return c.volume / 1000.0
    else
        # Note we use getfield, instead of getproperty
        # This is to avoid infinite recursion
        return getfield(a, v)
    end
end

# simulating setters
function Base.setproperty!(c::Cup, v::Symbol, value)
    if v == :volume_in_L
        vol_ml = value * 1000.0
        c.volume = min(vol_ml, c.capacity)
    else
        # Note we use setfield, instead of setproperty
        # This is to avoid infinite recursion
        setfield!(c, v, value)
    end
end
```

Simulating inheritance can be done like so:

```julia
mutable struct Mug <: Cup
    volume
    capacity
end
```

Because `Mug` subtypes `Cup`, all functions defined for `Cup` will work for any `Mug` object and indeed anything that is a subtype of a `Cup`. This is the multiple dispatch system at work. Those of you who have worked with S3 and S4 classes in R will recognize this. Although working with the object oriented features of R is considered [an advanced topic](http://adv-r.had.co.nz/S4.html).

Here, we've simulated inheritance and object methods but note that we had to redefine the struct fields (`volume` and `capacity`) again. While this is explicit, in the real world it can become annoyingly verbose.

Thankfully there are alternatives available. The first I came across was to use the [`Mixers.jl` package](https://github.com/rafaqz/Mixers.jl). It provides a set of macros to help simulate inheritance. 
The second is to use the macro system in Julia directly. I won't claim I fully understand macros - it's similar to macros in C, but as I currently understand it, different enough that the concepts are not completely transferrable.

Side-story: Macros
------------------

A workable, if naive, explanation is that wherever you see a macro being used it simply means "do a set of things here" - this "set of things" can include manipulating the code itself. This is called "metaprogramming" but I won't be going into further detail here.

The very simple example below defines a macro `@sayhello`. Wherever you use `@sayhello` in your code gets replaced with a call to the `println` function.

```julia
# Example taken from the docs:
# https://docs.julialang.org/en/v1/manual/metaprogramming/#man-macros-1
macro sayhello(name)
    return :( println("Hello, ", $name) )
end

# @sayhello("Rick")
# > Hello, Rick
```

Well hang on, why bother with macros? We can do the same thing with a function with the same effect:

```julia
function sayhello(name)
    println("Hello, ", $name)
end
```

The difference is in the parsing and execution of the code. Where normally code gets compiled then executed, macros allow for the ability to change the code *before* its compilation and execution. This allows for some stuff that gets... meta....

Back to the main story
----------------------

Alright, so the gist of the above is that macros modify code before they get run. Below is the `@def` macro which is now in fairly common usage.

```julia
macro def(name, definition)
    return quote
        macro $(esc(name))()
            esc($(Expr(:quote, definition)))
        end
    end
end
```

What the `@def` macro does is create another macro which inserts the specified code into where it is used. So if I state:

```julia
@def nevermore begin
    println("Quoth the Raven")
end

# The compiler will replace `@nevermore` with the specified println call
```

Meta, right? But how does this help with inheritance?

Rather than use it as an on-the-fly search and replace, we can specify a common set of struct fields with defined types...

```julia
@def cup_fields begin
    volume::Float64
    capacity::Float64
    has_handle::Bool
    owner::String
end
```

As before, the newly specified macro (`@cup_fields`) inserts the code defined within into wherever it is used.

Now, rather than typing out the fields for every subtype of `Cup`, I can simply use the `@cup_fields` macro.

```julia
mutable struct Cup
    @cup_fields
end

mutable struct Mug <: Cup
    @cup_fields
end

mutable struct ShotGlass <: Cup
    @cup_fields
end
```

To make the example concrete, what Julia ends up parsing and running is the below.

```julia
# This:

# mutable struct ShotGlass <: Cup
#     @cup_fields
# end

# Becomes:
mutable struct ShotGlass <: Cup
    volume::Float64
    capacity::Float64
    has_handle::Bool
    owner::String
end
```

Much less verbose!


That's it for now, but check back later as I plan to write up a similar post on DataFrames. Before you go, you might want to check out...


Some other bits and pieces
--------------------------

<table class="tg">
  <tr>
    <th class="tg-0pky"></th>
    <th class="tg-0pky">Python</th>
    <th class="tg-0pky">Julia</th>
  </tr>
  <tr>
    <td class="tg-0pky">Dict keys</td>
    <td class="tg-0pky">
        <pre lang="python">
if key in some_dict:
    print("The key was in the dict")
        </pre>
    </td>
    <td class="tg-0pky">
        <pre lang="julia">
if haskey(some_dict, key)
    print("The key was found!")
end
        </pre>
    </td>
  </tr>
  <tr>
    <td class="tg-0pky">Argument unpacking</td>
    <td class="tg-0pky">
        <pre lang="python">
def example(a, b, c):
    print(a, b, c)

inputs = [1, 2, 3]
example(*inputs)
        </pre>
    </td>
    <td class="tg-0pky">
        <pre lang="julia">
function example(a, b, c)
    print(a, b, c)
end

inputs = [1 2 3]
example(inputs...)
        </pre>
    </td>
  </tr>
  <tr>
    <td class="tg-0pky">Keyword argument unpacking</td>
    <td class="tg-0pky">
    <pre lang="python">
def example(a, b, c):
    print(a, b, c)

inputs = {a: 1, b: 2, c: 3}
example(**inputs)
    </pre>
    </td>
    <td class="tg-0pky">
      <pre lang="julia">
# Note the semi-colon!
# In Julia, keyword arguments have to be explicitly 
# declared. These are separated from regular 
# positional arguments with the semi-colon. 
function example(; a, b, c)
    print(a, b, c)
end

inputs = Dict(
    :a => 1,
    :b => 2,
    :c => 3
)
example(; inputs...)
      </pre>
    </td>
  </tr>
  <tr>
    <td class="tg-0pky">Ternary operator</td>
    <td class="tg-0pky"><pre lang="python">c = a if a > 0 else b</pre></td>
    <td class="tg-0pky"><pre lang="julia">c = a > 0 ? a : b</pre></td>
  </tr>
  <tr>
    <td class="tg-0pky">String concatenation</td>
    <td class="tg-0pky"><pre lang="python">
    str_a = "Hello"
    str_b = " World"
    str_c = str_a + str_b
    </pre></td>
    <td class="tg-0pky"><pre lang="julia">
    str_a = "Hello"
    str_b = " World"
    str_c = str_a * str_b
    </pre></td>
  </tr>
  <tr>
    <td class="tg-0pky">Iterating over a dictionary</td>
    <td class="tg-0pky"><pre lang="python">
    for k, v in dict:
        pass
    </pre></td>
    <td class="tg-0pky"><pre lang="julia">
    for (k, v) in dict
        (k, v)
    end
    # In Julia: `k in dict` will produce 
    # the key->value
    </pre></td>
  </tr>
</table>