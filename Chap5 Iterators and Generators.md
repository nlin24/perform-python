## Chap5 Iterators and Generators
* It's a good practice to have separate functions generating data(generator) and processing data(iterator), thus make them more general
* The function processing data could call the function generating data as an input
  exmaple:
  ```
  def fibonacci_naive():
      """
      Function generating data and processing lumped together into one function
      Hard to read
      """
      i, j = 0, 1
      count = 0
      while i <= 5000:
          if i % 2:
              count += 1
          i, j = j, i + j
      return count


  def fibonacci_transform():
      """
      Function processing data calls function generating data (fibonacci())
      Easy to understand
      """
      count = 0
      for f in fibonacci():
          if f >= 5000:
              break
          if f % 2:
              count += 1
      return count
  ```


### Generator
* Use _yield_ keyword
* Low impact on memory with large data


### Iterator
* Just your regular _for_ or _while_ loops

