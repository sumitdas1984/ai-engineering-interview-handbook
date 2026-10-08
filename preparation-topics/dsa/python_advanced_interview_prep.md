# Python Advanced Features — Interview Prep

A focused interview-preparation guide covering 13 commonly used Python topics in order.

For each topic:
1. What is it?
2. Why is it used?
3. Simple example
4. 2–3 interview questions

---

## 1. Comprehensions

### What is it?
Comprehensions provide a concise way to create collections such as lists, dictionaries, and sets from an iterable.

Common forms:

```python
[x * x for x in numbers]
{x: x * x for x in numbers}
{x * x for x in numbers}
```

### Why is it used?
- Makes simple transformation/filtering code shorter and more readable.
- Commonly used in Python coding interviews.
- Avoids writing a full loop for simple collection creation.

### Simple example
```python
numbers = [1, 2, 3, 4, 5]

squares = [x * x for x in numbers]
even = [x for x in numbers if x % 2 == 0]
```

### Interview questions
1. What is the difference between list comprehension and a normal `for` loop?
2. How would you create a list of squares of only the even numbers?
3. What is dictionary comprehension? Give an example.

---

## 2. Lambda Functions

### What is it?
A lambda is a small anonymous function, usually used when a simple function is needed temporarily.

Syntax:
```python
lambda arguments: expression
```

### Why is it used?
- Useful for short one-line functions.
- Frequently used with `sorted()`, `map()`, and `filter()`.
- Avoids defining a separate named function for very simple logic.

### Simple example
```python
people = [("John", 35), ("Alice", 25), ("Bob", 30)]

people.sort(key=lambda x: x[1])
print(people)
```

### Interview questions
1. What is a lambda function and when would you use it?
2. Can a lambda contain multiple statements?
3. How would you sort a list of dictionaries by a particular key using lambda?

---

## 3. `map()`, `filter()`, `reduce()`

### What is it?
- `map()` → transform each item.
- `filter()` → keep items satisfying a condition.
- `reduce()` → repeatedly combine items into one result.

### Why is it used?
They provide concise ways to transform, filter, and aggregate data.

### Simple example
```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

squares = list(map(lambda x: x * x, numbers))
even = list(filter(lambda x: x % 2 == 0, numbers))
total = reduce(lambda x, y: x + y, numbers)
```

### Interview questions
1. What is the difference between `map()`, `filter()`, and `reduce()`?
2. What does `map()` return in Python 3?
3. When would you prefer a list comprehension over `map()`?

---

## 4. Iterators

### What is it?
An iterator is an object that produces values one at a time using the iterator protocol:
`__iter__()` and `__next__()`.

### Why is it used?
- Allows sequential traversal of data.
- Supports lazy processing.
- Forms the foundation of Python's `for` loop.

### Simple example
```python
numbers = [10, 20, 30]

it = iter(numbers)

print(next(it))  # 10
print(next(it))  # 20
print(next(it))  # 30
```

When there are no more values, `next(it)` raises `StopIteration`.

### Interview questions
1. What is the difference between an iterable and an iterator?
2. What do `__iter__()` and `__next__()` do?
3. How does a `for` loop internally use an iterator?

---

## 5. Generators

### What is it?
A generator is a convenient way to create an iterator using `yield`. It produces values lazily, one at a time.

### Why is it used?
- Saves memory.
- Useful for large datasets and files.
- Enables lazy computation and streaming.

### Simple example
```python
def numbers():
    yield 1
    yield 2
    yield 3

for n in numbers():
    print(n)
```

Large-file example:
```python
def read_lines(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()
```

### Interview questions
1. What is the difference between `return` and `yield`?
2. Why are generators memory efficient?
3. What is the relationship between a generator and an iterator?

---

## 6. Decorators

### What is it?
A decorator is a function that wraps another function to add or modify behavior without changing the original function's code.

### Why is it used?
Common uses include logging, authentication, timing, caching, validation, and retry logic.

### Simple example
```python
def log_call(func):
    def wrapper():
        print("Function started")
        func()
        print("Function finished")
    return wrapper

@log_call
def hello():
    print("Hello")

hello()
```

Conceptually, `@log_call` is equivalent to `hello = log_call(hello)`.

### Interview questions
1. What is a decorator and why is it useful?
2. What does `@decorator` mean internally?
3. Why is `functools.wraps` commonly used when writing decorators?

---

## 7. `*args` and `**kwargs`

### What is it?
They allow a function to accept a variable number of arguments.

- `*args` → variable positional arguments.
- `**kwargs` → variable keyword arguments.

### Why is it used?
- Flexible function APIs.
- Common in decorators and wrapper functions.
- Useful when the number of arguments is unknown.

### Simple example
```python
def add(*args):
    return sum(args)

print(add(1, 2))
print(add(1, 2, 3, 4))
```

```python
def show_info(**kwargs):
    for key, value in kwargs.items():
        print(key, value)

show_info(name="Sumit", age=40)
```

### Interview questions
1. What is the difference between `*args` and `**kwargs`?
2. What are `args` and `kwargs` inside the function?
3. How are `*args` and `**kwargs` useful in decorators?

---

## 8. Advanced OOP

### What is it?
Important advanced Python OOP features include `@classmethod`, `@staticmethod`, `@property`, abstract classes, `@abstractmethod`, and magic/dunder methods.

### Why is it used?
These features support encapsulation, interfaces/contracts, object creation, utility methods, and custom object behavior.

### Simple example
```python
from abc import ABC, abstractmethod

class Animal(ABC):

    @abstractmethod
    def make_sound(self):
        pass

    @property
    @abstractmethod
    def species(self):
        pass

class Dog(Animal):

    def make_sound(self):
        return "Bark"

    @property
    def species(self):
        return "Canine"

dog = Dog()

print(dog.make_sound())
print(dog.species)
```

`@abstractmethod` defines a contract that subclasses must implement. `@property` lets a method be accessed like an attribute.

### Interview questions
1. What is the difference between `@staticmethod`, `@classmethod`, and an instance method?
2. What is an abstract class and why use `@abstractmethod`?
3. What is the purpose of `@property`?

---

## 9. Context Managers

### What is it?
A context manager manages setup and cleanup around a block of code, commonly using `with`.

### Why is it used?
It ensures resources are properly cleaned up even if an exception occurs.

Common examples: files, database connections, locks, and network resources.

### Simple example
```python
with open("data.txt") as f:
    data = f.read()
```

A custom context manager can implement:
```python
__enter__()
__exit__()
```

### Interview questions
1. What is a context manager and why is `with` useful?
2. What are `__enter__()` and `__exit__()`?
3. How would you create a custom context manager?

---

## 10. `collections`

### What is it?
The `collections` module provides specialized container data structures.

Important interview classes:
`Counter`, `defaultdict`, `deque`, `namedtuple`.

### Why is it used?
These structures simplify common coding problems and can be more expressive or efficient than manually implementing the same behavior.

### Simple example
```python
from collections import Counter

words = ["a", "b", "a", "c", "a"]
count = Counter(words)

print(count)
# Counter({'a': 3, 'b': 1, 'c': 1})
```

```python
from collections import defaultdict

groups = defaultdict(list)
groups["fruit"].append("apple")
groups["fruit"].append("banana")
```

```python
from collections import deque

q = deque([1, 2, 3])
q.append(4)
q.popleft()
```

### Interview questions
1. When would you use `Counter` instead of a normal dictionary?
2. What is the advantage of `defaultdict`?
3. Why use `deque` instead of a list for queue operations?

---

## 11. `itertools`

### What is it?
`itertools` provides efficient iterator-building functions for common iteration patterns.

Important functions include `chain`, `combinations`, `permutations`, `product`, and `groupby`.

### Why is it used?
- Avoids complicated nested loops.
- Provides lazy iterator-based operations.
- Useful for combinations, permutations, and Cartesian products.

### Simple example
```python
from itertools import combinations

numbers = [1, 2, 3]

print(list(combinations(numbers, 2)))
# [(1, 2), (1, 3), (2, 3)]
```

```python
from itertools import chain

print(list(chain([1, 2], [3, 4])))
# [1, 2, 3, 4]
```

### Interview questions
1. What is `itertools.combinations()` used for?
2. What is the difference between `combinations()` and `permutations()`?
3. Why are many `itertools` functions memory efficient?

---

## 12. `functools`

### What is it?
`functools` provides utilities for working with functions and callable objects.

Important features include `lru_cache`, `cache`, `partial`, `reduce`, and `wraps`.

### Why is it used?
- Function caching/memoization.
- Creating partially applied functions.
- Preserving function metadata in decorators.
- Functional programming utilities.

### Simple example
```python
from functools import lru_cache

@lru_cache
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))
```

`partial` example:
```python
from functools import partial

def multiply(x, y):
    return x * y

double = partial(multiply, 2)

print(double(5))  # 10
```

### Interview questions
1. What is `lru_cache` and when would you use it?
2. What does `functools.wraps` do in a decorator?
3. What is `functools.partial`?

---

## 13. Async Python and Concurrency

### What is it?
Python provides several ways to execute work concurrently, including `threading`, `multiprocessing`, `concurrent.futures`, and `asyncio`.

For asynchronous programming, key concepts are `async`, `await`, coroutines, and `asyncio`.

### Why is it used?
Concurrency is useful for I/O-bound work such as HTTP/API calls, database calls, file/network I/O, and multiple LLM API calls.

### Simple example
```python
import asyncio

async def task(name):
    print(f"Starting {name}")
    await asyncio.sleep(2)
    print(f"Finished {name}")

async def main():
    await asyncio.gather(
        task("A"),
        task("B")
    )

asyncio.run(main())
```

The two tasks can make progress concurrently while waiting.

Basic interview mental model:

```text
I/O-bound → async/asyncio or threads
CPU-bound → multiprocessing
```

This is a simplified rule; the right choice depends on the workload and libraries involved.

### Interview questions
1. What is the difference between synchronous and asynchronous code?
2. What do `async` and `await` mean?
3. When would you use `asyncio` vs threading vs multiprocessing?

---

# Quick Revision Sheet

| # | Topic | Key thing to remember |
|---|---|---|
| 1 | Comprehensions | Concise collection creation |
| 2 | Lambda | Small anonymous function |
| 3 | map/filter/reduce | Transform / filter / aggregate |
| 4 | Iterators | `iter()` + `next()` |
| 5 | Generators | `yield` + lazy evaluation |
| 6 | Decorators | Add behavior by wrapping functions |
| 7 | `*args/**kwargs` | Variable arguments |
| 8 | Advanced OOP | Properties, abstract classes, class/static methods |
| 9 | Context Managers | Resource setup/cleanup with `with` |
| 10 | collections | `Counter`, `defaultdict`, `deque` |
| 11 | itertools | Efficient iterator utilities |
| 12 | functools | Caching, partial functions, decorators |
| 13 | Async/concurrency | Handle I/O/parallel work efficiently |

## Interview Priority

### Must know very well
- Comprehensions
- Lambda
- `map()` / `filter()`
- Iterators
- Generators
- Decorators
- `*args` / `**kwargs`
- Advanced OOP
- Async/concurrency

### Know reasonably well
- Context managers
- `collections`
- `itertools`
- `functools`

For coding interviews, don't just memorize definitions. For each topic, be able to **write a small example from scratch and explain when you would use it**.
