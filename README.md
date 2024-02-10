# Oreilly High Performance Python, 2nd Edition My Notes


## Chap 2 Profiling a Script to Find Bottlenecks


## Measure Script Elapsed Time
* Use Python [_timeit_](https://docs.python.org/3/library/timeit.html) module to time small bit of Python code
  ```
  python -m timeit "'-'.join(str(n) for n in range(100))"
  10000 loops, best of 5: 30.2 usec per loop

  python -m timeit "'-'.join([str(n) for n in range(100)])"
  10000 loops, best of 5: 27.5 usec per loop

  python -m timeit "'-'.join(map(str, range(100)))"
  10000 loops, best of 5: 23.2 usec per loop
  ```


* Use Unix _time_ command to measure script elapsed time
  ```
  $ /usr/bin/time -p python julia1_nopil.py
  Length of x: 1000
  Total elements: 1000000
  calculate_z_serial_purepython took 8.279886722564697 seconds
  real 8.84
  user 8.73
  sys 0.10
  ```
  * Not the shell _time_ command
  * **real** records the elapsed time
  * **user* records the amount of CPU time spent on the task out side of kernel functions
  * **sys** records time spent in kernel level functions
  * **real** - **user** - **sys** = time spent waiting for I/O
  * Set verbose to get more information
  * **Major (requiring I/O) page faults** inidicates when data is not in RAM and OS needs to load pages of data from disk


### Measure CPU Usage
* Use Python _cProfile_ module to measure the time taken to run every function
  ```
  $ python -m cProfile -s cumulative julia1_nopil.py
  ...
  Length of x: 1000
  Total elements: 1000000
  calculate_z_serial_purepython took 11.498265266418457 seconds
           36221995 function calls in 12.234 seconds

    Ordered by: cumulative time

    ncalls  tottime  percall  cumtime  percall filename:lineno(function)
         1    0.000    0.000   12.234   12.234 {built-in method builtins.exec}
         1    0.038    0.038   12.234   12.234 julia1_nopil.py:1(<module>)
         1    0.571    0.571   12.197   12.197 julia1_nopil.py:23
                                                (calc_pure_python)
         1    8.369    8.369   11.498   11.498 julia1_nopil.py:9
                                                (calculate_z_serial_purepython)
  34219980    3.129    0.000    3.129    0.000 {built-in method builtins.abs}
   2002000    0.121    0.000    0.121    0.000 {method 'append' of 'list' objects}
         1    0.006    0.006    0.006    0.006 {built-in method builtins.sum}
         3    0.000    0.000    0.000    0.000 {built-in method builtins.print}
         2    0.000    0.000    0.000    0.000 {built-in method time.time}
         4    0.000    0.000    0.000    0.000 {built-in method builtins.len}
         1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
  ```
  * **-s cumulative** sort the function calls in decending orders by cumulative time
  * **_lsprof** is the old name of the cProfile module
  * Visualizing cProfile Output with the Pytho``````````````````n _SnakeViz_ module5if needed


* Use Python _line_profiler_ for line by line measurements
  * The _line_profiler_ module profiles a function at a time and requires adding a decorator (@profile) to the function in question
  * Once function decorated, run the _kerprof_ command in the console.
    ```
    $ kernprof -l -v julia1_lineprofiler.py
    ...
    Wrote profile results to julia1_lineprofiler.py.lprof
    Timer unit: 1e-06 s

    Total time: 49.2011 s
    File: julia1_lineprofiler.py
    Function: calculate_z_serial_purepython at line 9

    Line #      Hits  Per Hit   % Time  Line Contents
    ==============================================================
        9                              @profile
        10                              def calculate_z_serial_purepython(maxiter,
                                                                        zs, cs):
        11                            """Calculate output list using Julia update rule"""
        12         1   3298.0      0.0      output = [0] * len(zs)
        13   1000001      0.4      0.8      for i in range(len(zs)):
        14   1000000      0.4      0.7          n = 0
        15   1000000      0.4      0.9          z = zs[i]
        16   1000000      0.4      0.8          c = cs[i]
        17  34219980      0.5     38.0          while abs(z) < 2 and n < maxiter:
        18  33219980      0.5     30.8              z = z * z + c
        19  33219980      0.4     27.1              n += 1
        20   1000000      0.4      0.8          output[i] = n
        21         1      1.0      0.0      return output
    ```
    * **-l** measures line by line
    * **-v** turns on verbose output to the console, else output is saved to **.lprof** that can analyzed by the *line_profile* module


### Measure Memory Usage
* Memory profiling is not as straightforward as CPU usage profile
* Memory may not be allocated the deterministic amount as observed from outside the process
* Observing the gross trend is likely to lead to better insight than observing the behavior of the memory allocation by code line by line
* Use Python _memory_profiler_ module. This makes the script running 10 - 100 times slower
* Installing the Python _psutil_ module may help _memory_profiler_ to run quicker but still pretty slow
```
* _memory_profiler_ requires adding a decorator (@profile) to the function it's profiling
$ python -m memory_profiler julia1_memoryprofiler.py
...

Line #    Mem usage    Increment   Line Contents
================================================
     9  126.363 MiB  126.363 MiB   @profile
    10                             def calculate_z_serial_purepython(maxiter,
                                                                     zs, cs):
    11                                 """Calculate output list using...
    12  133.973 MiB    7.609 MiB       output = [0] * len(zs)
    13  136.988 MiB    0.000 MiB       for i in range(len(zs)):
    14  136.988 MiB    0.000 MiB           n = 0
    15  136.988 MiB    0.000 MiB           z = zs[i]
    16  136.988 MiB    0.000 MiB           c = cs[i]
    17  136.988 MiB    0.258 MiB           while n < maxiter and abs(z) < 2:
    18  136.988 MiB    0.000 MiB               z = z * z + c
    19  136.988 MiB    0.000 MiB               n += 1
    20  136.988 MiB    0.000 MiB           output[i] = n
    21  136.988 MiB    0.000 MiB       return output

...

Line #    Mem usage    Increment   Line Contents
================================================
    24   48.113 MiB   48.113 MiB   @profile
    25                             def calc_pure_python(draw_output,
                                                        desired_width,
                                                        max_iterations):
    26                                 """Create a list of complex...
    27   48.113 MiB    0.000 MiB       x_step = (x2 - x1) / desired_width
    28   48.113 MiB    0.000 MiB       y_step = (y1 - y2) / desired_width
    29   48.113 MiB    0.000 MiB       x = []
    30   48.113 MiB    0.000 MiB       y = []
    31   48.113 MiB    0.000 MiB       ycoord = y2
    32   48.113 MiB    0.000 MiB       while ycoord > y1:
    33   48.113 MiB    0.000 MiB           y.append(ycoord)
    34   48.113 MiB    0.000 MiB           ycoord += y_step
    35   48.113 MiB    0.000 MiB       xcoord = x1
    36   48.113 MiB    0.000 MiB       while xcoord < x2:
    37   48.113 MiB    0.000 MiB           x.append(xcoord)
    38   48.113 MiB    0.000 MiB           xcoord += x_step
    44   48.113 MiB    0.000 MiB       zs = []
    45   48.113 MiB    0.000 MiB       cs = []
    46  125.961 MiB    0.000 MiB       for ycoord in y:
    47  125.961 MiB    0.258 MiB           for xcoord in x:
    48  125.961 MiB    0.512 MiB               zs.append(complex(xcoord, ycoord))
    49  125.961 MiB    0.512 MiB               cs.append(complex(c_real, c_imag))
    50
    51  125.961 MiB    0.000 MiB       print("Length of x:", len(x))
    52  125.961 MiB    0.000 MiB       print("Total elements:", len(zs))
    53  125.961 MiB    0.000 MiB       start_time = time.time()
    54  136.609 MiB   10.648 MiB       output = calculate_z_serial...
    55  136.609 MiB    0.000 MiB       end_time = time.time()
    56  136.609 MiB    0.000 MiB       secs = end_time - start_time
    57  136.609 MiB    0.000 MiB       print(calculate_z_serial_purepython...
    58
    59  136.609 MiB    0.000 MiB       assert sum(output) == 33219980
```
* We could visualize the data by calling _memory_profiler_'s _mprof_ utility
```
mprof run julia1_memoryprofiler.py
```
```
mprof plot
```


### Introspect running Python process
* Use the Python _py-spy_ module to instrospect an existing Python process
* It's console _top_-like
* No performance impact to the running process
* Install via ```pip install py-spy ```
* Needs to first identify the process identifier PID using _ps_
```
$ ps -A -o pid,rss,cmd | ack python
...
15953 96156 python julia1_nopil.py
...
$ sudo env "PATH=$PATH" py-spy top --pid 15953
```
* You can also visualize it in flame chart via  ```py-spy --flame profile.svg -- python julia1_nopil.py```


### Inspect the Python Script Bytecode
* Use Python _dis_ module to inspect the bytecode executed by the script

    code example:
    ```
    import dis

    class Foo(object):
    def __init__(self):
        pass

    def foo(self, x):
        return x + 1

    def bar():
    x = 5
    y = 7
    z = x + y
    return z

    def main():
    dis.dis(bar) # disassembles `bar`
    dis.dis(Foo) # disassembles each method in `Foo`
    ```

    output:
    ```
    14           0 LOAD_CONST               1 (5)
                3 STORE_FAST               0 (x)

    15           6 LOAD_CONST               2 (7)
                9 STORE_FAST               1 (y)

    16          12 LOAD_FAST                0 (x)
                15 LOAD_FAST                1 (y)
                18 BINARY_ADD
                19 STORE_FAST               2 (z)

    17          22 LOAD_FAST                2 (z)
                25 RETURN_VALUE

    Disassembly of __init__:
    8           0 LOAD_CONST               0 (None)
                3 RETURN_VALUE

    Disassembly of foo:
    11           0 LOAD_FAST                1 (x)
                3 LOAD_CONST               1 (1)
                6 BINARY_ADD
                7 RETURN_VALUE
    ```
* First column describes the line number in the script
* **>>** indicates jump points to other places in the code
* Third column is the address of the bytecode instruction
* Fourth column is the operation name
* Fifth column is the parameter for the operation; the index of the argument in the code block’s name and constant table.
* Last column is annotation helping lining up the bytecode with the original Python parameters
* The more lines of bytecode will execute more slowly than fewer equivalent lines of bytecode that used by Python built-in functions
  
  for an example:
  ```
  def fn_expressive(upper=1_000_000):
      total = 0
      for n in range(upper):
          total += n
      return total

  def fn_terse(upper=1_000_000):
      return sum(range(upper))

  assert fn_expressive() == fn_terse(), "Expect identical results from both functions"
  ```

  timeit:
  ```
  In [2]: %timeit fn_expressive()
  52.4 ms ± 86.4 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

  In [3]: %timeit fn_terse()
  18.1 ms ± 1.38 ms per loop (mean ± std. dev. of 7 runs, 100 loops each)
  ```

  Bytecode representations:
  ```
  In [4]: import dis
  In [5]: dis.dis(fn_expressive)
    2           0 LOAD_CONST               1 (0)
                2 STORE_FAST               1 (total)

    3           4 SETUP_LOOP              24 (to 30)
                6 LOAD_GLOBAL              0 (range)
                8 LOAD_FAST                0 (upper)
               10 CALL_FUNCTION            1
               12 GET_ITER
          >>   14 FOR_ITER                12 (to 28)
               16 STORE_FAST               2 (n)

    4          18 LOAD_FAST                1 (total)
               20 LOAD_FAST                2 (n)
               22 INPLACE_ADD
               24 STORE_FAST               1 (total)
               26 JUMP_ABSOLUTE           14
          >>   28 POP_BLOCK

    5     >>   30 LOAD_FAST                1 (total)
               32 RETURN_VALUE

  In [6]: dis.dis(fn_terse)
    8           0 LOAD_GLOBAL              0 (sum)
                2 LOAD_GLOBAL              1 (range)
                4 LOAD_FAST                0 (upper)
                6 CALL_FUNCTION            1
                8 CALL_FUNCTION            1
              10 RETURN_VALUE
  ```