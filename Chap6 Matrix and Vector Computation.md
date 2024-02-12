## Chap6 Matrix and Vector Computation
* Vector computation solves several calculations at once
* Use _numpy_ to speed up the code


## Use Python _numpy_ module
* __numpy__ object gives memory locality and vectorized operations for numerical computations
* __numpy__ arrays store data in contiguous chunks of memory and suppor vectorized oeprations on its data
* __numpy__ vector calculation utilizes finely tuned and specially built objects for dealing with arrays of numbers
* __numpy__ implmentation is faster than pure Python looping methods and list comprehension methods
* __numpy__ supports in-place operations and can reduce memory allocations
  exmample:
  ```
  >>> import numpy as np
  >>> array1 = np.random.random((10,10))
  >>> array2 = np.random.random((10,10))
  >>> id(array1)
  140199765947424  1
  >>> array1 += array2  //numpy array in-place operations
  >>> id(array1)
  140199765947424  2
  >>> array1 = array1 + array2
  >>> id(array1)
  140199765969792  1   // numpy array out-of-place oepration, array re-assigned
  ```
* When the __numpy__ array sizes are small (smaller than the CPU cache size), out-of-place oeprations are faster than in-place operations because it can benefit from __numpy__array vectorizations
* When __numpy__ array sizes grow bigger than CPU cache size, then in-place operations are faster because of fewer allocations and cache misses
  example:
  ```
  >>> import numpy as np

  >>> %%timeit array1, array2 = np.random.random((2, 100, 100))  1 3
  ... array1 = array1 + array2
  6.45 µs ± 53.3 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)

  >>> %%timeit array1, array2 = np.random.random((2, 100, 100))  1
  ... array1 += array2
  5.06 µs ± 78.7 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)

  >>> %%timeit array1, array2 = np.random.random((2, 5, 5))  2
  ... array1 = array1 + array2
  518 ns ± 4.88 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)

  >>> %%timeit array1, array2 = np.random.random((2, 5, 5))  2
  ... array1 += array2
  1.18 µs ± 6 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
  ```
* Performance comparisons example between Python list, Python list comprehensions, Python arrays, and __numpy__ arrays:
  ```
  from array import array
  import numpy


  def norm_square_list(vector):
      """
      >>> vector = list(range(1_000_000))
      >>> %timeit norm_square_list(vector)
      85.5 ms ± 1.65 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
      """
      norm = 0
      for v in vector:
          norm += v * v
      return norm


  def norm_square_list_comprehension(vector):
      """
      >>> vector = list(range(1_000_000))
      >>> %timeit norm_square_list_comprehension(vector)
      80.3 ms ± 1.37 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
      """
      return sum([v * v for v in vector])


  def norm_square_array(vector):
      """
      >>> vector_array = array('l', range(1_000_000))
      >>> %timeit norm_square_array(vector_array)
      101 ms ± 4.69 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
      """
      norm = 0
      for v in vector:
          norm += v * v
      return norm


  def norm_square_numpy(vector):
      """
      >>> vector_np = numpy.arange(1_000_000)
      >>> %timeit norm_square_numpy(vector_np)
      3.22 ms ± 136 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
      """
      return numpy.sum(vector * vector)  1


  def norm_square_numpy_dot(vector):
      """
      >>> vector_np = numpy.arange(1_000_000)
      >>> %timeit norm_square_numpy_dot(vector_np)
      960 µs ± 41.1 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
      """
      return numpy.dot(vector, vector)  2
  ```


### Use Python __Pandas__ for tabular data process
* __Panda__ is the data manipulation tool in the scientific Python ecosystem
* Supports a tabular datatype with rows and columns called **DataFrames**
* Operations on a DataFrame apply to all cells in the row/column
* Install the optional dependencies __numexpr__ and __bottleneck__ for additional performance improvements


### Use Linux _perf_ tool to get insight on program executions 
* We can use the Linux console _perf_ command to get insight on how the CPU runs a program
  example:
  ```
  $ perf stat -e cycles,instructions,\
    cache-references,cache-misses,branches,branch-misses,task-clock,faults,\
    minor-faults,cs,migrations python diffusion_python_memory.py


  Performance counter stats for 'python diffusion_python_memory.py':

    415,864,974,126      cycles                    #    2.889 GHz                      
  1,210,522,769,388      instructions              #    2.91  insn per cycle           
        656,345,027      cache-references          #    4.560 M/sec                    
        349,562,390      cache-misses              #   53.259 % of all cache refs      
    251,537,944,600      branches                  # 1747.583 M/sec                    
      1,970,031,461      branch-misses             #    0.78% of all branches          
      143934.730837      task-clock (msec)         #    1.000 CPUs utilized          
              12,791      faults                    #    0.089 K/sec                  
              12,791      minor-faults              #    0.089 K/sec                  
                117      cs                        #    0.001 K/sec                  
                  6      migrations                #    0.000 K/sec                  

      143.935522122 seconds time elapsed
  ```
  * **task-clock**: how many clock cycles the task took
  * **instructions**: how many CPU instructions the code issued
  * **cycles**: how many CPU cycles it took to run all of these instructions
  * **cs** or **context switches** : when happens, the program's execution is halted and another program is allowed to run isntead, for an example, to wait for a kernel operation to finish (such as I/O for disk, memory, or network) 
  * **migrations**: describe when a program iexecution is halted and then  moved to another CPU core at context switching
  * **faults** or **page-fault**: happens when the program reqeusts data from a device (disk, network, etc) that hasnt' been read into memory yet
  * **minor-faults**: interrupts threw by the OS to pause a program at the beginning of its execution and properly allocates the memory. This is called a **lazy allocation system**
  * **cache-references**: this metric tracks nubmer of times when the programm successfully reference to a data that is in the CPU cache
  * **cache-miss**: data referenced is not in CPU cache and needed to be fetch from memory
  * **branches**: tracks a time in the code where the execution flow changes, such as iff...else statements
  * **branch-misses**: tracks when CPU misses a branch prediction
