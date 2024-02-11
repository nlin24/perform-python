## Chap 3 List and Tuples


* _Lists_ and _tuples_ are a class of the _array_ data structure
* _Lists_ are dynamic arrays that contents can be modified and resized
* _Tuples_ are static arrays that contents are fixed and immutable
* **Numpy** arrays, on the other hand, hold static data and can store only that type of data
* Both lists and tuples can take mixed types


### List
* The Python _list_ data structure by default stores the system memory location reference to the actual data
* Searching in a sorted _list_ has better performance than unosrted _list_ (for an example, binary search algorithm)
* Use object's magic functions __eq__ and __lt__ for comparison.
* Python has a built-in sorting algorithm **Tim sort**
* Python has a built-in module for adding elements to a _list_ in order. [**bisect**](https://docs.python.org/3/library/bisect.html).
  Example:
  ```
  import bisect
  import random
  # important_numbers will already be in order because we inserted new elements
  # with bisect.insort
  important_numbers = []
  for i in range(10):
      new_number = random.randint(0, 1000)
      bisect.insort(important_numbers, new_number)
  print(important_numbers)
  # > [14, 265, 496, 661, 683, 734, 881, 892, 973, 992]
  ```
* _List_ reserves extra spaces in order to stay efficient when appending new data. For an example, a _list_ of size 100,000,000 created with any append operation actually uses 112,500,007 elements’ worth of memory


### Tuples
* _Tuples_ do not support **append** operation; two lists can only be concatenated and creating a new list
* A tuple holding 100,000,000 data will only ever use exactly 100,000,000 elements’ worth of memory. This makes tuples lightweight and preferable when data becomes static
* Tuples are cached by the Python runtime and data no longer needed are not given back to the system (garbage collected), which means that we don’t need to talk to the kernel to reserve memory every time we want to use one
