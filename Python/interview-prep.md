# Python — Interview Prep

## Table of Contents
- [1. Python Basics](#1-python-basics)
- [2. Functions & Scope](#2-functions--scope)
- [3. OOP in Python](#3-oop-in-python)
- [4. Iterators, Generators & Comprehensions](#4-iterators-generators--comprehensions)
- [5. Decorators](#5-decorators)
- [6. Error Handling](#6-error-handling)
- [7. Modules & Packages](#7-modules--packages)
- [8. Common Standard Library & Built-ins](#8-common-standard-library--built-ins)
- [9. Concurrency](#9-concurrency)
- [10. Pythonic Code](#10-pythonic-code)

---

## 1. Python Basics

**Q: What kind of language is Python?**

Python is:
- **Interpreted** — code is executed line by line by the Python interpreter (CPython by default), not compiled to machine code beforehand.
- **Dynamically typed** — variable types are determined at runtime, not declared at compile time. A variable can hold any type and can be reassigned to a different type.
- **Garbage collected** — memory is managed automatically via reference counting plus a cyclic garbage collector.
- **High-level and general-purpose** — used for web development, scripting, data science, automation, ML, and more.
- **Duck typed** — if it walks like a duck and quacks like a duck, it's treated as a duck. You don't need explicit interfaces; you just need the right methods.

---

**Q: Mutable vs immutable types — what's the difference?**

- **Immutable** — once created, the value cannot be changed. A new object is created on modification.
  - Examples: `int`, `float`, `str`, `tuple`, `frozenset`, `bytes`
- **Mutable** — the object can be changed in place.
  - Examples: `list`, `dict`, `set`, `bytearray`, custom class instances

```python
# Immutable: s is rebound to a new object
s = "hello"
s += " world"   # new string object; original "hello" unchanged

# Mutable: list is modified in place
lst = [1, 2, 3]
lst.append(4)   # same list object, now [1, 2, 3, 4]
```

This matters when passing to functions: mutable objects can be modified inside the function (passed by object reference).

---

**Q: List vs Tuple vs Set vs Dict?**

| | List | Tuple | Set | Dict |
|---|---|---|---|---|
| Ordered | Yes | Yes | No | Yes (insertion order, Python 3.7+) |
| Mutable | Yes | No | Yes | Yes |
| Duplicates | Yes | Yes | No | Keys: No, Values: Yes |
| Syntax | `[1,2,3]` | `(1,2,3)` | `{1,2,3}` | `{"a":1}` |
| Use when | General sequence | Fixed data, hashable | Uniqueness, membership tests | Key-value mapping |

Sets and dict keys must be **hashable** (immutable types). Lists cannot be set elements or dict keys.

---

## 2. Functions & Scope

**Q: What are first-class functions and higher-order functions?**

In Python, **functions are first-class objects** — they can be assigned to variables, passed as arguments, and returned from other functions.

A **higher-order function** is one that takes another function as an argument or returns a function.

```python
def apply(func, value):   # higher-order function
    return func(value)

result = apply(str.upper, "hello")   # passing a function as argument
print(result)   # "HELLO"
```

Built-in examples: `map()`, `filter()`, `sorted(key=...)`.

---

**Q: What are `*args` and `**kwargs`?**

- **`*args`** — collects extra positional arguments into a tuple.
- **`**kwargs`** — collects extra keyword arguments into a dict.

```python
def greet(*args, **kwargs):
    for name in args:
        print(f"Hello, {name}")
    for key, val in kwargs.items():
        print(f"{key} = {val}")

greet("Alice", "Bob", role="engineer", city="NYC")
```

They allow functions to accept a variable number of arguments. Order must be: regular params → `*args` → keyword-only params → `**kwargs`.

---

**Q: What are lambda functions?**

Anonymous, single-expression functions defined inline with the `lambda` keyword.

```python
square = lambda x: x ** 2
print(square(5))   # 25

# Common use: as a sort key
people = [("Alice", 30), ("Bob", 25)]
people.sort(key=lambda p: p[1])   # sort by age
```

Use them for short, throwaway functions. For anything more complex, use a named `def`.

---

**Q: What is the LEGB scope rule?**

Python resolves variable names in this order:
1. **L**ocal — inside the current function.
2. **E**nclosing — in the enclosing function's scope (for nested functions/closures).
3. **G**lobal — at the module/file level.
4. **B**uilt-in — Python's built-in names (`len`, `print`, `range`, etc.).

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        print(x)   # finds "enclosing" — Enclosing scope
    inner()

outer()
```

Use `global x` to modify a global variable inside a function. Use `nonlocal x` to modify an enclosing variable inside a nested function.

---

## 3. OOP in Python

**Q: How do you define a class in Python?**

```python
class Animal:
    species = "Unknown"   # class variable (shared by all instances)

    def __init__(self, name, age):   # constructor
        self.name = name             # instance variables
        self.age = age

    def speak(self):
        print(f"{self.name} makes a sound")

dog = Animal("Rex", 3)
dog.speak()
```

`self` is a reference to the current instance. It must be the first parameter of every instance method (by convention named `self`).

---

**Q: How does inheritance work in Python?**

```python
class Dog(Animal):   # Dog inherits from Animal
    def speak(self):   # override
        print(f"{self.name} says Woof")

    def fetch(self):
        print("Fetching!")

class GuideDog(Dog):
    def __init__(self, name, age, owner):
        super().__init__(name, age)   # call parent constructor
        self.owner = owner
```

Python supports **multiple inheritance**: `class C(A, B)`. The **MRO (Method Resolution Order)** — determined by the C3 linearization algorithm — defines which method is called when the same method exists in multiple parents. Check it with `ClassName.__mro__`.

---

**Q: What are dunder (magic) methods?**

Special methods with double underscores that Python calls implicitly for built-in operations.

| Method | Called when |
|---|---|
| `__init__` | Object is created |
| `__str__` | `str(obj)` or `print(obj)` |
| `__repr__` | `repr(obj)`, developer-facing string |
| `__len__` | `len(obj)` |
| `__eq__` | `obj1 == obj2` |
| `__lt__`, `__gt__` | `<`, `>` comparisons |
| `__add__` | `obj1 + obj2` |
| `__getitem__` | `obj[key]` |
| `__iter__`, `__next__` | Iteration protocol |
| `__enter__`, `__exit__` | `with` statement (context manager) |

```python
class Point:
    def __init__(self, x, y): self.x, self.y = x, y
    def __str__(self): return f"Point({self.x}, {self.y})"
    def __add__(self, other): return Point(self.x + other.x, self.y + other.y)
```

---

**Q: What are `@property`, `@staticmethod`, and `@classmethod`?**

- **`@property`** — turns a method into a read-only attribute. Add `.setter` for write access. Lets you add logic to attribute access without changing the interface.

```python
class Circle:
    def __init__(self, radius): self._radius = radius

    @property
    def radius(self): return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0: raise ValueError("Radius cannot be negative")
        self._radius = value
```

- **`@staticmethod`** — a method that belongs to the class namespace but doesn't receive `self` or `cls`. It's a plain function that happens to live in the class. Use for utility functions related to the class.

- **`@classmethod`** — receives `cls` (the class itself) instead of `self`. Used for alternative constructors or factory methods.

```python
class Date:
    def __init__(self, y, m, d): self.y, self.m, self.d = y, m, d

    @classmethod
    def from_string(cls, s):   # alternative constructor
        y, m, d = map(int, s.split("-"))
        return cls(y, m, d)

d = Date.from_string("2024-03-15")
```

---

## 4. Iterators, Generators & Comprehensions

**Q: What is the difference between an iterable and an iterator?**

- **Iterable** — an object you can loop over. Has `__iter__()` which returns an iterator. Examples: `list`, `str`, `dict`, `range`.
- **Iterator** — an object that produces values one at a time via `__next__()`. Maintains state. Raises `StopIteration` when exhausted.

`for x in iterable` implicitly calls `iter(iterable)` to get an iterator, then calls `next()` repeatedly.

```python
lst = [1, 2, 3]          # iterable
it = iter(lst)            # iterator
print(next(it))           # 1
print(next(it))           # 2
```

---

**Q: What are generators? How does `yield` work?**

A generator is a function that uses `yield` to produce a sequence of values lazily — one at a time, pausing execution between yields. It returns a generator object (which is an iterator).

```python
def count_up(n):
    for i in range(n):
        yield i   # pauses here, resumes on next()

gen = count_up(3)
print(next(gen))   # 0
print(next(gen))   # 1

for val in count_up(5):
    print(val)
```

**Why use generators?** They are memory efficient — they don't compute the whole sequence upfront. Critical for large datasets or infinite sequences.

Generator expressions: `(x**2 for x in range(10))` — like list comprehensions but lazy.

---

**Q: What are comprehensions?**

Concise syntax for building collections.

```python
# List comprehension
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]

# Dict comprehension
word_lengths = {word: len(word) for word in ["hello", "world"]}

# Set comprehension
unique_lengths = {len(word) for word in ["hi", "hello", "hey"]}

# Generator expression (lazy)
total = sum(x**2 for x in range(1000000))   # no intermediate list in memory
```

Prefer comprehensions over manual loops when building a collection — they are more readable and often faster.

---

## 5. Decorators

**Q: What is a decorator and how does it work?**

A decorator is a function that **wraps another function** to add behavior before and/or after it runs, without modifying the original function's code.

```python
def log(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Done")
        return result
    return wrapper

@log   # equivalent to: greet = log(greet)
def greet(name):
    print(f"Hello, {name}")

greet("Alice")
# Calling greet
# Hello, Alice
# Done
```

Use `functools.wraps(func)` inside the wrapper to preserve the original function's metadata (`__name__`, `__doc__`).

**Common use cases:** logging, timing, authentication/authorization checks, caching (`@functools.lru_cache`), retrying failed calls.

Decorators can be stacked — they apply bottom-up:
```python
@decorator_a
@decorator_b
def func(): ...
# equivalent to: func = decorator_a(decorator_b(func))
```

---

## 6. Error Handling

**Q: How does exception handling work in Python?**

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")       # caught specific exception
except (TypeError, ValueError):
    print("Type or value error")
else:
    print("No exception occurred")   # runs only if no exception
finally:
    print("Always runs")             # cleanup — runs regardless
```

- `except Exception` catches all non-system-exiting exceptions. Avoid bare `except:` — it catches everything including `KeyboardInterrupt`.
- `finally` is for cleanup code (closing files, releasing locks) that must run regardless of outcome.
- `else` is for code that should only run if the `try` block succeeded — keeps the happy path clean.

---

**Q: How do you define custom exceptions?**

```python
class AppError(Exception):
    pass

class ValidationError(AppError):
    def __init__(self, field, message):
        self.field = field
        super().__init__(f"Validation failed on '{field}': {message}")

try:
    raise ValidationError("email", "invalid format")
except ValidationError as e:
    print(e.field)   # "email"
    print(e)         # "Validation failed on 'email': invalid format"
```

Inherit from `Exception` (or a more specific built-in) rather than `BaseException`.

---

## 7. Modules & Packages

**Q: How does the Python import system work?**

- A **module** is a single `.py` file. A **package** is a directory containing `__init__.py` (and modules/subpackages).
- `import module_name` — imports the module; access with `module_name.something`.
- `from module import name` — imports a specific name directly into scope.
- `from package.submodule import func` — import from nested packages.

Python searches for modules in: current directory → directories in `PYTHONPATH` → standard library → site-packages (installed packages).

`__init__.py` makes a directory a package. It can be empty or can expose a clean API for the package.

---

**Q: What are virtual environments and why use them?**

A virtual environment is an **isolated Python environment** with its own interpreter and site-packages. This prevents dependency conflicts between projects.

```bash
python -m venv venv          # create
source venv/bin/activate     # activate (macOS/Linux)
pip install requests         # install into this env only
pip freeze > requirements.txt   # save dependencies
pip install -r requirements.txt # reproduce env elsewhere
deactivate                   # exit env
```

Always use a virtual environment per project. Never install packages globally unless they're tools (like `pipx`-managed CLIs).

---

## 8. Common Standard Library & Built-ins

**Q: What are some essential built-in functions?**

```python
map(func, iterable)          # apply func to each element → lazy iterator
filter(func, iterable)       # keep elements where func returns True → lazy iterator
zip(a, b)                    # pair up elements from two iterables
enumerate(iterable)          # yields (index, value) pairs
sorted(iterable, key=...)    # returns a new sorted list
reversed(sequence)           # reverse iterator
any(iterable)                # True if at least one element is truthy
all(iterable)                # True if all elements are truthy
isinstance(obj, type)        # type checking
```

```python
names = ["Charlie", "Alice", "Bob"]
for i, name in enumerate(names):
    print(i, name)

pairs = list(zip([1, 2, 3], ["a", "b", "c"]))   # [(1,'a'), (2,'b'), (3,'c')]

print(any([False, False, True]))   # True
print(all([True, True, False]))    # False
```

---

**Q: What are useful standard library modules?**

- **`collections`** — `Counter` (count occurrences), `defaultdict` (dict with default values), `deque` (double-ended queue, O(1) popleft), `namedtuple` (lightweight immutable class).
- **`itertools`** — `chain`, `islice`, `product`, `combinations`, `permutations`, `groupby` — combinatorial and infinite iterators.
- **`functools`** — `lru_cache` (memoization), `partial` (partial function application), `reduce`, `wraps`.

```python
from collections import Counter, defaultdict
from functools import lru_cache

word_count = Counter("abracadabra")   # Counter({'a': 5, 'b': 2, ...})

graph = defaultdict(list)
graph["A"].append("B")   # no KeyError if "A" doesn't exist

@lru_cache(maxsize=None)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)
```

---

## 9. Concurrency

**Q: What is the GIL and why does it matter?**

The **GIL (Global Interpreter Lock)** is a mutex in CPython that allows only **one thread to execute Python bytecode at a time**, even on multi-core machines. This means Python threads do not achieve true CPU parallelism for CPU-bound tasks.

**Why it exists:** simplifies CPython's memory management (reference counting).

**Impact:**
- **I/O-bound tasks** (network calls, file reads) — threads still help because the GIL is released during I/O waits. `threading` is useful here.
- **CPU-bound tasks** (computation) — threads won't speed things up due to the GIL. Use `multiprocessing` instead (separate processes, each with its own GIL).

---

**Q: Threading vs Multiprocessing vs Asyncio — when to use each?**

| | `threading` | `multiprocessing` | `asyncio` |
|---|---|---|---|
| Best for | I/O-bound tasks | CPU-bound tasks | I/O-bound, high concurrency |
| True parallelism | No (GIL) | Yes (separate processes) | No (single thread) |
| Memory | Shared | Separate (IPC needed) | Shared |
| Overhead | Low | High (process spawn) | Very low |
| Complexity | Medium | Medium-High | Medium |

```python
# asyncio — for many concurrent I/O operations (e.g., many API calls)
import asyncio

async def fetch(url):
    await asyncio.sleep(1)   # non-blocking wait
    return f"data from {url}"

async def main():
    results = await asyncio.gather(fetch("url1"), fetch("url2"))
    print(results)

asyncio.run(main())
```

Rule of thumb: use `asyncio` for many concurrent I/O tasks (web scraping, APIs), `threading` for simple I/O concurrency, `multiprocessing` for CPU-heavy parallel computation.

---

## 10. Pythonic Code

**Q: What is a context manager and how does it work?**

A context manager handles **setup and teardown** automatically using the `with` statement. On enter, `__enter__` is called; on exit (normal or exception), `__exit__` is called.

```python
with open("file.txt", "r") as f:
    data = f.read()
# file is automatically closed here, even if an exception occurred

# Custom context manager
class DatabaseConnection:
    def __enter__(self):
        self.conn = connect_db()
        return self.conn

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()
        return False   # don't suppress exceptions

# Using contextlib for simple cases
from contextlib import contextmanager

@contextmanager
def managed_resource():
    resource = acquire()
    try:
        yield resource
    finally:
        release(resource)
```

---

**Q: What are type hints and why use them?**

Type hints annotate variables and function signatures with expected types. They don't enforce types at runtime (Python is still dynamically typed) but enable **static analysis tools** (mypy, pyright) and improve code readability.

```python
def greet(name: str, times: int = 1) -> str:
    return (f"Hello, {name}\n") * times

from typing import Optional, Union, List, Dict, Tuple

def find_user(user_id: int) -> Optional[str]:   # may return None
    ...

def process(items: List[int]) -> Dict[str, int]:
    ...
```

Use `from __future__ import annotations` for forward references, or use `|` syntax in Python 3.10+: `int | None` instead of `Optional[int]`.

---

**Q: What are common Pythonic idioms worth knowing?**

```python
# Swap variables without temp
a, b = b, a

# Ternary expression
value = "even" if n % 2 == 0 else "odd"

# Unpack a list/tuple
first, *rest = [1, 2, 3, 4]   # first=1, rest=[2,3,4]
first, *middle, last = [1, 2, 3, 4]

# Check membership in O(1) with set
valid = {1, 2, 3}
if x in valid: ...

# Dict .get() with default
name = user.get("name", "Anonymous")

# String joining (not +=)
parts = ["Hello", "World"]
sentence = " ".join(parts)

# Use enumerate instead of range(len(...))
for i, val in enumerate(items):
    print(i, val)

# Use zip to iterate pairs
for a, b in zip(list1, list2):
    ...

# f-strings for formatting (Python 3.6+)
msg = f"Hello, {name}! You are {age} years old."
```

Being "Pythonic" means writing clear, concise code that follows Python conventions — preferring readability, using built-ins, and avoiding unnecessary boilerplate.
