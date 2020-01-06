---
title: Some thoughts on a new language stack for integrated modelling
---

As a PhD candidate, a core part of my research is the development of Integrated (Environmental) Models. This involves a whole gamut of work from data engineering and analysis, development of software to applying machine learning techniques. Much of this requires sufficient domain knowledge to integrate the models from the various disciplines involved.

Often a minimum viable prototype can be quickly mocked up in a language like Python or R - and indeed in some cases these may be sufficient enough for its purpose. The issue arises when conducting model calibration, sensitivity, uncertainty and exploratory analysis as the model may be run 10s or 100s of thousands of times, potentially even millions. Runtime of 1 second may be adequate for demonstration or purposeful application. But analysis of a model taking a second to run means over a day of runtime if 100,000 runs are necessary. I know of no integrated model that incorporates models across multiple disciplines that completes in 1 second. Minutes, maybe. Hours, certainly.

This isn't about inefficient programming practice or poor software design. It's largely about the complexity of the task being conducted and the methods used to develop the models and interop between them, which at times are limited by historic context. Models are often developed in R or Python, but extensions to these and and external code written in lower-level languages (often C or Fortran) are not rare.

Concurrency and multiprocessing isn't leveraged often due to the (perceived or real) complexity associated with it. On the method side, it seems that file-based feed-forwarding - where the output of one model is written out and read back in as input to another - is the most common method that I come across (to be clear, this is my own subjective observation). This perhaps harkens back to the historic context.

Using files for data interop may be the only method available if you're working with a model from 1976, the code for which is written in FORTRAN IV, has no comments and is filled with GOTOs. Other times it's just the most pragmatic option as each individual model is developed by a separate person or team, perhaps for a different purpose altogether. It may be too costly or too time-consuming to develop bespoke interfaces between all the models involved, even with the aid of integration frameworks such as OpenMI. Fully integrated multi-system models that are purpose built are few and far between for this reason.

Here comes the crux of this post: Given the context (integrated modelling across a variety of disciplines and languages), the requirements of analysing model performance, and as a consequence, the importance of minimizing runtime, what mix of programming languages is ideal for future scientific model development?

I've been weighing up some options for a language to add to my toolbox for some time now. I've been a (relatively) long time Python user but in certain cases it doesn't provide the "oomph" that is needed. Concurrent and parallel processing, while doable, isn't as straightforward as it could be with many pitfalls. So far I've been happy with the easy pathways to fast programs that are available in Python, but each approach has a limitation.

* Numba does provide a speed boost with minimal code changes, but usually only for numeric or numpy-heavy code and it's not fully compatible with all of the Python language spec. In my experience the speed boost obtained is less than that of Cython.

* I'm most comfortable with Cython, but while it's easy to apply, it is not advisable to use it everywhere (and nor should you).

* I could write an extension in C++ or, if I'm feeling particularly adventurous, Fortran but for various reasons (usually time) I usually don't want to do that.

The strategy with all of the above is to target specific hotspots - parts of your program causing the most slowdown - to get you to a point where the program overall is "fast enough" for your purpose. Experienced R users would be familiar with the `.C()` function and the `Rcpp` package for much the same purpose.

In some cases however, it's desirable to have the entire program faster than what Python can achieve; although admittedly those cases have been few and far between - for now. Currently when such cases do occur, I can only bite the bullet and make do with what can be done with Numba or Cython, or otherwise port across the slow parts into an entirely different language.

I've often said that a good generalist programmer is comfortable with at least two, maybe three, languages and knowledgeable in more. A high-level language and a low-level language, with an optional language in between. For myself, this was Python, Cython, with the occassional C and Fortran (going from high to low of course).

In the sciences knowing just a high-level language often results in long runtimes. I've seen and heard of colleagues waiting hours, days, and even weeks for a fairly small handful of model runs to complete. At the other end, having only a low-level language in your toolbox often results in long development times and buggy software all to achieve something that could be done in a few minutes in Python or R - which quickly becomes obsolete as new requirements are fleshed out.

For a long time the combination of Python and Cython suited my needs. But I am fast approaching the limits of what these can provide. This doesn't mean I'm abandoning Python - it will remain my primary hammer for the foreseeable future. But this begs the question: where to from here?

**Runtime speed** is a growing concern, but "traditional" low-level languages can be a pain to work with. To give some social/organisational context, I'm expected to develop models and code with people who may not know what a compiler is, let alone why anyone would use anything other than R.

For this reason, ability to **interoperate** across languages is just as important, if only to avoid the crude file-based data passing method where possible.

**Verbosity** is a concern because believe it or not, I don't want to spend a lot of time writing out code. Sure, a good IDE will solve the problem but I don't like reading verbose code either. A language that supports generics and templating is then ideal so as to avoid repetitive boilerplate code as much as possible.

**Readability** of the code matters too - I want to understand what my younger self wrote as quickly as possible. If readability is enforced as a design of the language as much as possible, then that's a plus. 

**Cross-platform**, or at least Windows and Linux support, as development often occurs on Windows machines, and the models could be run on HPC platforms, including supercomputers.

Easy **multiprocessing and concurrency**. If these are treated as first-class features of the language, that would be a major plus.

Finally, **Development speed**. I want to get a minimum viable prototype up and running quickly with a minimum of fuss, and then move that to full development with little cognitive overhead. Rapid development is a key reason why Python and R have made such in-roads in the corporate/research worlds, especially in the recent explosion in data science/engineering. A "nice to have" would be quick compilation times (cos who wants to sit around waiting for that?).

Which two to three languages would best complement each other in the research/modelling context? 

In the next post I'll talk briefly about some potential candidates, along with some (very unscientific) benchmarks.
