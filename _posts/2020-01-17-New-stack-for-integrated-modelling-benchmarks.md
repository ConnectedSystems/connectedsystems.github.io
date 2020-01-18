---
title: A new language stack for integrated modelling - Some unscientific benchmarks
---

In the previous post I spoke of finding a new language to include in my toolbox. Here, I'll briefly introduce the ones I've looked at. Before I begin, note that I sometimes add the suffix "lang" to the names of programming languages (e.g. `R` vs `Rlang`). This is a now widespread convention as the name may have other common usages (useful especially in Google searches).

You may ask why not stick with any one of the existing languages, like C++ or Fortran?

There are many reasons why I want to avoid these languages. The importance of languages like C, C++ and Fortran is undisputable. However, they are all several decades old - C++ was initially released in 1979 , C in 1972, and Fortran in 1954. With this age comes historic baggage. 

Keep in mind there is **no perfect language**. Only relative scales of "good for what you want to do". In my context, I'm after a simple language with first class support for multiprocessing and, above all, high readability and reduced verbosity [note: Yes, I am lumping concurrency with parallelism for the purpose of this post.]

These older languages tend to be more verbose than I would like, and treat parallelisation in all its forms as an afterthought. Understandably so however, as when these languages were developed, most computers only had a single core.

I'll share one pertinent quote form Ian Joyner, author of "Objects Unencapsulated: Java, Eiffel, and C++" and "C++? A critique of C++ and programming and language trends of the 1990s" who wrote in 1996 ([link to PDF](http://www.quinn.echidna.id.au/Quinn/C++-Critique-3ed.pdf)):

    "Concurrency requires much cleaner languages, than the single processor languages of today." (p 41)

You may find many other pertinent quotes with respect to programming language use and design, despite it being more than two decades old now. 

Still on C++, Jon Harrop provides an insightful answer on its flaws ([link to Quora](https://qr.ae/TSqvAS) - sign up may be required). The gist of both above is that C++ is a large, complicated, language with many features and many ways to achieve the same results, with a syntax that adds to the confusion. 

An example shared by [MadhavenRP on Stackoverflow](https://stackoverflow.com/a/4027301) illustrates this with respect to pointers.

In my view Fortran suffers from this too, though perhaps to a lesser extent. By all accounts Fortran 2018 is a decent language but I still wouldn't want to use it for general programming purposes. A big issue is that you cannot mix any one of the Fortran standards in common use (i.e. any one of Fortran 77, 90, 95, 2003, 2008, 2018) at least not without issue. That said, one way interaction between a newer spec and an older spec (e.g. F2008 using F90 code) is possible as the vast majority of the language is backwards compatible, much like with C and C++. 

Personally I dislike Fortran syntax as I find it too verbose and hampers readability. Property and method accessors are indicated by a percent sign for example, which looks awkward to me:

```
# Python
circle.area()

# C++
circle->area()

# Fortran
circle%area()
```

Enter Julia (2018), Go (2012), Nim (2019) and V (v1.0 not yet released).

In my first post I spoke of learning **Julia**, which has the potential to become the new Python/R/Matlab. In my current experience applying it, I've found it enjoyable to use albeit with small instances of frustration to be expected when one is learning a new language.

It is a gradually typed language with a Just-in-Time (JIT) compilation system that can, in some if not most cases, achieve speeds comparable to Fortran and C. Another key feature in my mind is the near zero overhead interoperability with existing languages including C, Fortran, R and Python. In its most recent release, Julia developers have added automatic multithreading support (albeit currently in an experimental form) - parallelising a `for` loop is dead simple and straight forward.

So far this is the strongest contender to replace both Python and R from my toolbox.

Lower level languages include **Go**, a language developed by engineers at Google (v1.0 release in 2012). The creators of the language include many notable computer scientists and engineers, but the one that sticks out in my mind is
[Ken Thompson](https://en.wikipedia.org/wiki/Ken_Thompson), who created the B programming language (the predecessor to C). It is now well regarded as being a simple language for engineers to pick up and apply quickly. Its strengths include fast compilation times and concurrency as a first class citizen.

Of the ahead-of-time compiled languages discussed here, Golang is likely to be the most long-lived having corporate support and wide adoption.

Apart from Golang, there are two wild cards: **Nim** and **V**. Both are very young languages even compared to Julia and Golang. Version 1.0 of Nim was released in Sept of 2019, and I consider V as not having even reached beta stage (more on this later), currently on v0.1.23 with many promised but currently unavailable features.

Nim has Python-like syntax with C-like performance. It supports multiple compiler backends and so acts as a transpiler of sorts, currently capable of converting Nim code into C, C++, Objective-C, or Javascript. An advantage of compiling to one of the established languages is the ability to leverage all the compiler optimizations that have been developed over the past few decades as well as being compatible with all the existing tooling for those languages.

Vlang is syntactically similar to Go and markets itself as a "simple language for building maintainable programs". It claims to be able to compile directly to native binaries and supports interop with C.

The advantage of their youth can be seen in their respective syntax. 
Compare examples of concurrent code in Fortran, Go, and Julia courtesy of Rosetta Code (see [here](https://rosettacode.org/wiki/Concurrent_computing)).

In short, the Fortran implementation using the OpenMP library is 36 lines (comments not included, but blank lines are). Incidentally, the proprietary Intel compiler can automatically thread simple loops (as described [here](https://software.intel.com/en-us/articles/threading-fortran-applications-for-parallel-performance-on-multi-core-systems)) but being proprietary, it is not freely available for use.

The same concurrent program in Golang is 23 lines, and in Julia it is just 10 lines (written for v0.6 and verified to work under v1.3) both of which require no 3rd party library.

You be the judge as to which language is most readable.

So, a simple benchmark.

I've chosen to write the same nested loop in the standard implementations of Julia, Golang, Nim and V, and in Python, R, C and Fortran to give a sense of its syntax.

The Python code is:

```python
# Python 3.7.1

a = 10

for n in range(1, 10000001):
    result = 1
    for i in range(1, a+1):
        result = result + (a - i) + n
    # End for
# End for

print(result)
# Output: 100000046
```

Total runtimes were captured using [ptime](http://www.pc-tools.net/win32/) with the sole exception of Vlang which I could not get working on Windows. Timings for V were instead taken using `time` via Windows Subsystem for Linux. In this case I use the timing as reported under the `Real` column.

For Python I also wrote examples using Numba and Cython to showcase the ease at which improvements can be made.

For Julia I also profiled the execution time for comparison purposes as its JIT has a warmup cost with the first run compiling code. Timings for total, initial, and secondary runs will be reported.

All timings were conducted on my laptop with a Core i7-8550u.
Timings are best out of three. Before jumping to conclusions, please read the **Other Notes** section below.

You can go see the code for yourself [here](https://github.com/ConnectedSystems/loop-lang-comparison).


**High-Level Languages**

| Language (version) 	| Timing (seconds) 	| Comments/Notes 	|
|---------------------------	|------------------	|-----------------------------------------------------------------------	|
| Python (v3.7.3) 	| 17.960 	|  	|
| Python w/ Numba (v0.46.0) 	| 0.99 	| Added two lines!  	|
| Cython (v0.29.13) 	| 0.142 	|  	|
| R (v3.6.1) 	| 6.984 	|  	|
| Julia - total runtime (v1.3) 	| 0.287 	| Difference between total and initial run timings is due to JIT warmup 	|
| Julia - initial run 	| 0.043597 	| Using internal `@time` macro 	|
| Julia - subsequent runs 	| 0.003426 	| as above 	|


**Ahead-of-Time compiled languages**

| Language (version) 	| Timing (seconds) 	| Comments/Notes 	|
|--------------------	|------------------	|-------------------------------------------------------------------	|
| C 	| 0.036 	| Compiled using `gcc` v4.7.0 with `-O2` flag 	|
| Fortran 	| 0.043 	| Compiled using `gfortran` v4.7.0 with `-O2` flag 	|
| Go (v1.13.5) 	| 0.109 	| `go build loop.go` |
| V (v0.1.23) 	| 0.168 	| `v ./loop.v` |
| Nim (v1.0.4) 	| 0.033 	| Flags used: `-d:release --passC:-flto --passL:-s --gc:markAndSweep` 	|

**Other notes**

Huge performance gains from just adding [two lines](https://github.com/ConnectedSystems/loop-lang-comparison/blob/master/Python/nbloop.py) to Python code to use Numba. Even bigger gains from adding type information for Cython and compiling. Cython holds up pretty well all things considered in my opinion.

Nimlang's syntax is very similar to Python - it could almost be a drop-in replacement. Overall, I'm impressed with Nim. 

All compiled languages, with the exception of Go, are within spitting distance of each other.

Generally speaking I found Go to have the most intuitive syntax.
The syntax for V was inspired by Go so these two are similar.

That said, the early stage of development for V is very evident, I ran into several issues getting it working. It looks to be a promising language (some have gone so far as to call it "what Go could have been") but it's too early for any extensive use.

Julia's JIT warmup is fairly significant and from experience gets worse as more code and packages are used. Depending on what packages are included, this can be seconds.

JIT compilation to bytecode was added to R in v3.4 which significantly speeds up looping compared to v2.x. There's also compilers which targets the JVM ([Renjin](https://www.renjin.org/) and [GraalVM](https://www.graalvm.org/)) but I've yet to try these.

## A side-story on R

While R is a very capable language, I never fully got onboard just simply because of its many quirks. It always seems to be planning something devious to catch me by surprise: "ah-ha! Gotcha!".

I'm relatively comfortable with R but even for this simple benchmark I got caught out momentarily. Printing out the result directly outputs an incorrect value.

```R
# Seemingly incorrect value!
print(result)
# [1] 1e+08

print(as.integer(result))
# [1] 100000046
```

This reminded me of my "favourite" R gotcha:

```R
> is.integer(1)
[1] FALSE
```

What's happening here is that R stores everything as a `double` underneath unless you specify that you specifically want an integer by adding `L` to the value.

You can see this in action below:

```R
is.integer(1)
# [1] FALSE
# Above is false because `1` is actually a double `1.0`

is.integer(1L)
# [1] TRUE
# `L` indicates you explicitly want an integer.
```

So three solutions here:

Either explicitly cast the result to an integer using `as.integer()` or add `L` to the initial values so subsequent operations are all done on integers. The third approach is to provide a value to the `print` function to explictly tell R to show all digits: `print(result, digits=20)`. By default, R only shows the first 7 after which it "helpfully" truncates it.

In the end I went with the first approach as it was the first I tried. Interestingly I saw a performance regression when using explicit integer types (the code slowed by a second or so). I'm assuming that R is optimized for calculations with floats.

## Final thoughts

Nim is looking to be "a better Cython" as you can write [Python extensions in Nim](https://github.com/yglukhov/nimpy). In fact, I may well end up using it as such for both Python and Julia.

I find the syntax for Golang enjoyable, but wary of its verbosity and repetitious nature as it lacks generics. A common complaint is having to write out essentially the same thing over and over with small differences to handle different data types. It's also lacking common tooling for data science purposes (perhaps unsurprising given its focus on corporate/business/infrastructure development) so at best it will be used for very specific purposes.

I'm impressed so far with Julia. While its JIT warmup hampers its use for short single use cases, if multiple (long) runs can be conducted then it's very possible for it to outperform C and Fortran.

Possible language trifecta:

Julia and/or Python for high-level stuff

Nim or Go for lower-level stuff although I'm leaning more to Nim for this. Certainly Nim is looking to be an attractive alternative to Cython in cases where speed is critical.
