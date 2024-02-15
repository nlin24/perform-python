## Chap7 Compiling to C
* Cython compiles to C. It's a new hybrid language type of Python and C
* Numba is speicialized for numpy code only
* PyPy, a just-in-time compiler for non-numpy for normal Python executables
* Compiled Python code that tends to run faster with mathematical and lots of loops that repeat the same operation
* Code that calls out to external libraries (such as regular expressions, string operations, and calls to database libraries) is unlikely to show any speedup 
* Programs that are I/O-bound are also unlikely to show significant speedup


### Just-In-Time Compilers vs. Ahead-Of-Ttime Compilers
* AOT compiling creates a static library taht's specialized to your machine and instantly be sued to work
* JIT let the compiler decide to compile just the part of the code at runtime, and not much work up front. JIT is not suitable for short but frequently running scripts


### Cython
* Skipped. Not going to use unless I need to


### [Pypy](https://www.pypy.org/)
* PyPy may not help with:
  * Short-running processes that take less than few seconds; the JIT compiler won't have enough time to warm up
  * If all the time is spent in run-time libraries (i.e. in C functions), and not actually running Python code, the JIT compiler will not help
* Can run in parallel with multiprocessing
PyPy example:
```
$ pypy3
Python 3.6.1 (784b254d6699, Apr 14 2019, 10:22:42)
[PyPy 7.1.1-beta0 with GCC 6.2.0 20160901] on linux
Type "help", "copyright", "credits", or "license" for more information.
And now for something completely different
```
Another example:
```
$ pypy3 -m ensurepip
Collecting setuptools
Collecting pip
Installing collected packages: setuptools, pip
Successfully installed pip-9.0.1 setuptools-28.8.0

$ pip3 install ipython
Collecting ipython

$ ipython
Python 3.6.1 (784b254d6699, Apr 14 2019, 10:22:42)
Type 'copyright', 'credits', or 'license' for more information
IPython 7.8.0 -- An enhanced Interactive Python. Type '?' for help.
```