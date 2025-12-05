# Python Programming Cheatsheet (Beginner → Advanced)

Target: **Python 3** (examples work in Python 3.10+).

---

## 1. Basics

### 1.1 Variables and Data Types

- Dynamic typing: the type is attached to the value, not the variable name.
- Common built-in scalar types: `int`, `float`, `str`, `bool`.

```python
x = 10          # int
pi = 3.14       # float
name = "Ada"    # str
is_active = True  # bool (capital T/F)

print(type(x))       # <class 'int'>
print(type(pi))      # <class 'float'>
print(type(name))    # <class 'str'>
print(type(is_active))  # <class 'bool'>
```

**Tip:** `bool` is a subclass of `int` (`True == 1`, `False == 0`). Avoid arithmetic with booleans for clarity.

### 1.2 Type Conversion and Checking

```python
# Conversions
x = "42"
num = int(x)          # '42' -> 42
pi_str = str(3.14)    # 3.14 -> '3.14'

# Safe conversion pattern
value = "not_a_number"
try:
    num2 = float(value)
except ValueError:
    num2 = None

# Checking types
print(type(num) is int)        # True
print(isinstance(num, int))    # Preferred

# isinstance works with inheritance
print(isinstance(True, int))   # True
```

**Gotcha:** `type(obj) is SomeType` does not allow subclasses; `isinstance(obj, SomeType)` does.

### 1.3 Comments

```python
# Single-line comment
x = 10  # Inline comment

"""Multi-line strings are often used as multi-line comments.
They are actually string literals that are just not used.
Commonly used for docstrings (first statement in a module/function/class)."""

def add(a, b):
    """Return the sum of a and b."""
    return a + b
```

**Tip:** Use docstrings (`"""..."""`) for public functions/classes/modules.

### 1.4 Input/Output Operations

```python
# Input always returns a string
name = input("Enter your name: ")  # e.g. user types: Ada
age = int(input("Enter your age: "))

# Basic printing
print("Hello", name, "you are", age)

# Custom separators and endings
print("A", "B", "C", sep=", ")     # A, B, C
print("Line without newline", end="...")
print(" then this")

# f-strings (formatted string literals)
print(f"{name} will be {age + 1} next year")
```

---

## 2. Operators

### 2.1 Arithmetic, Comparison, Logical, Bitwise

```python
a, b = 7, 3

# Arithmetic
print(a + b)   # 10
print(a - b)   # 4
print(a * b)   # 21
print(a / b)   # 2.333... (float division)
print(a // b)  # 2 (floor division)
print(a % b)   # 1 (modulo)
print(a ** b)  # 343 (power)

# Comparison (all return bool)
print(a == b)  # False
print(a != b)  # True
print(a > b)   # True
print(a >= b)  # True
print(a < b)   # False

# Logical
x, y = True, False
print(x and y)  # False
print(x or y)   # True
print(not x)    # False

# Bitwise (on integers)
print(a & b)    # 3  -> 0b111 & 0b011 = 0b011
print(a | b)    # 7  -> 0b111 | 0b011 = 0b111
print(a ^ b)    # 4  -> 0b111 ^ 0b011 = 0b100
print(a << 1)   # 14 -> shift left
print(a >> 1)   # 3  -> shift right
```

### 2.2 Assignment and Identity Operators

```python
# Assignment variations
x = 5
x += 2   # 7
x *= 3   # 21
x -= 1   # 20
x /= 2   # 10.0 (note: now float)

# Identity
a = [1, 2]
b = a
c = [1, 2]

print(a == c)  # True  (same contents)
print(a is c)  # False (different objects)
print(a is b)  # True  (same object)
```

**Gotcha:** Use `is`/`is not` for identity comparisons (e.g. `x is None`), not for general equality.

### 2.3 Membership Operators

```python
nums = [1, 2, 3]
print(2 in nums)       # True
print(4 not in nums)   # True

text = "hello"
print("e" in text)     # True
print("ell" in text)   # True
```

---

## 3. Data Structures

### 3.1 Lists

- Ordered, mutable, can contain mixed types.

```python
# Creation
nums = [1, 2, 3]
mixed = [1, "two", 3.0]
from_iterable = list(range(5))  # [0, 1, 2, 3, 4]

# Indexing & slicing
print(nums[0])      # 1
print(nums[-1])     # 3 (last element)
print(nums[1:3])    # [2, 3]
print(nums[:2])     # [1, 2]
print(nums[::2])    # [1, 3] (step)

# Common methods
nums.append(4)
nums.extend([5, 6])
nums.insert(1, 1.5)
print(nums)  # [1, 1.5, 2, 3, 4, 5, 6]

nums.remove(1.5)     # removes first matching value
last = nums.pop()    # pops last element (6)

nums.sort()          # in-place sort
sorted_nums = sorted(nums, reverse=True)  # returns new list

# Copying lists
copy1 = nums.copy()      # shallow copy
copy2 = nums[:]          # slicing copy
```

**Gotcha:** `list_of_lists = [[0] * 3] * 3` creates *three references* to the same inner list. Use a comprehension instead: `[[0] * 3 for _ in range(3)]`.

### 3.2 Tuples

- Ordered, **immutable**.

```python
coords = (10, 20)
print(coords[0])      # 10

# Packing & unpacking
point = (3, 4)
x, y = point
print(x, y)           # 3 4

# Single-element tuple
single = (42,)
not_tuple = (42)
print(type(single))     # <class 'tuple'>
print(type(not_tuple))  # <class 'int'>
```

Common use: returning multiple values from functions, dictionary keys, fixed records.

### 3.3 Sets

- Unordered, unique elements, mutable.

```python
fruits = {"apple", "banana", "orange"}

# Empty set
empty_set = set()   # {} is an empty dict, not a set

# Operations
more_fruits = {"banana", "kiwi"}
print(fruits | more_fruits)   # union
print(fruits & more_fruits)   # intersection
print(fruits - more_fruits)   # difference
print(fruits ^ more_fruits)   # symmetric difference

# Methods
fruits.add("pear")
fruits.discard("banana")   # no error if missing

print("apple" in fruits)   # membership test
```

### 3.4 Dictionaries

- Mapping of keys to values.

```python
person = {
    "name": "Ada",
    "age": 36,
    "languages": ["Python", "C"]
}

print(person["name"])          # 'Ada'
print(person.get("age", 0))    # 36
print(person.get("city", "N/A"))

# Iteration
for key, value in person.items():
    print(key, "->", value)

# Methods
person["age"] = 37
person["city"] = "London"

age = person.pop("age")  # removes and returns value

keys = list(person.keys())
values = list(person.values())
items = list(person.items())
```

**Tip:** Keys must be hashable (e.g. `str`, `int`, `tuple` of immutables). Lists and dicts cannot be keys.

### 3.5 Strings

- Immutable sequences of Unicode characters.

```python
s = "  Hello, World!  "

print(s.lower())         # '  hello, world!  '
print(s.strip())         # 'Hello, World!'
print(s.replace("World", "Python"))

parts = s.strip().split(",")  # ['Hello', ' World!']
joined = "-".join(["2025", "12", "05"])  # '2025-12-05'

# Indexing & slicing
print(s[2:7])            # 'Hello'
print(s[::-1])           # reversed string

# f-strings and formatting
name = "Ada"
age = 36
print(f"{name} is {age} years old")
print(f"Pi is approximately {3.14159:.2f}")  # 2 decimal places
```

**Gotcha:** Because strings are immutable, operations like concatenation in loops can be slow. Prefer `"".join(list_of_strings)`.

---

## 4. Control Flow

### 4.1 if / elif / else

```python
x = 10

if x > 10:
    print("Greater than 10")
elif x == 10:
    print("Exactly 10")
else:
    print("Less than 10")
```

### 4.2 Ternary Operator

```python
age = 20
status = "adult" if age >= 18 else "minor"
print(status)  # 'adult'
```

### 4.3 for and while Loops

```python
# for loop over iterable
for i in range(3):
    print(i)  # 0, 1, 2

# while loop
count = 3
while count > 0:
    print(count)
    count -= 1
```

### 4.4 break, continue, pass

```python
for i in range(5):
    if i == 2:
        continue   # skip 2
    if i == 4:
        break      # stop loop
    print(i)       # prints 0, 1, 3

# pass: placeholder where a statement is syntactically required
for i in range(3):
    pass  # TODO: implement later
```

### 4.5 Loop else Clause

```python
# 'else' runs if loop completes without 'break'

nums = [1, 3, 5, 7]

for n in nums:
    if n % 2 == 0:
        print("Found even number", n)
        break
else:
    print("No even numbers found")
```

---

## 5. Functions

### 5.1 Definition and Calling

```python
def greet(name):
    """Return a greeting message."""
    return f"Hello, {name}!"

msg = greet("Ada")
print(msg)
```

### 5.2 Parameters

```python
def describe_person(name, age=0, *hobbies, **extra):
    print(f"Name: {name}")          # positional or keyword
    print(f"Age: {age}")            # default value
    print("Hobbies:", hobbies)      # *args: tuple of extra positional args
    print("Extra info:", extra)     # **kwargs: dict of extra keyword args

# Example call
describe_person(
    "Ada", 36, "music", "math",
    city="London", famous_for="computing"
)

# Keyword-only arguments (after a bare *)
def connect(host, port, *, timeout=30, use_ssl=True):
    ...

connect("example.com", 443, timeout=10)
```

### 5.3 Return Values

```python
def min_max(values):
    return min(values), max(values)  # returns a tuple

mn, mx = min_max([3, 1, 4])
print(mn, mx)  # 1 4
```

### 5.4 Lambda Functions

```python
# Short, anonymous functions
square = lambda x: x * x
print(square(5))  # 25

words = ["banana", "apple", "cherry"]
# Sort by length, then alphabetically
words.sort(key=lambda w: (len(w), w))
print(words)
```

**Tip:** Use lambdas for very simple functions; for anything non-trivial, prefer `def` for readability.

### 5.5 Decorators (Basics)

```python
import time

def timing(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        duration = time.time() - start
        print(f"{func.__name__} took {duration:.4f}s")
        return result
    return wrapper

@timing
def slow_add(a, b):
    time.sleep(0.5)
    return a + b

print(slow_add(2, 3))
```

**Gotcha:** Without `functools.wraps`, decorators hide the original function's metadata (`__name__`, docstring). Use:

```python
from functools import wraps

def timing(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        ...
    return wrapper
```

---

## 6. Object-Oriented Programming

### 6.1 Classes and Objects

```python
class Dog:
    def __init__(self, name):
        self.name = name  # instance attribute

    def bark(self):
        print(f"{self.name} says woof!")

fido = Dog("Fido")
fido.bark()
```

### 6.2 Instance and Class Variables

```python
class Counter:
    total_counters = 0  # class variable (shared)

    def __init__(self):
        Counter.total_counters += 1
        self.value = 0   # instance variable

c1 = Counter()
c2 = Counter()

print(c1.total_counters)  # 2
print(c2.total_counters)  # 2
```

**Gotcha:** Mutable class variables (like lists) are shared by all instances. Usually better to initialize them in `__init__`.

### 6.3 Methods: instance, class, static

```python
class MathUtils:
    factor = 2  # class variable

    def __init__(self, base):
        self.base = base

    def scale(self):        # instance method
        return self.base * self.factor

    @classmethod
    def set_factor(cls, value):  # class method
        cls.factor = value

    @staticmethod
    def add(a, b):          # static method
        return a + b

m = MathUtils(10)
print(m.scale())       # 20
MathUtils.set_factor(3)
print(m.scale())       # 30
print(MathUtils.add(2, 3))
```

### 6.4 Inheritance and super()

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        raise NotImplementedError

class Cat(Animal):
    def __init__(self, name, color):
        super().__init__(name)
        self.color = color

    def speak(self):
        return "meow"

c = Cat("Milo", "black")
print(c.name, c.color, c.speak())
```

### 6.5 Magic / Dunder Methods

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"  # unambiguous

    def __str__(self):
        return f"({self.x}, {self.y})"        # user-friendly

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __len__(self):
        return 2

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)     # (4, 6)
print(repr(v1))
print(len(v1))
```

Common dunder methods: `__init__`, `__repr__`, `__str__`, `__len__`, `__eq__`, `__lt__`, `__iter__`, `__getitem__`, `__enter__`, `__exit__`, etc.

---

## 7. Error Handling

### 7.1 try / except / else / finally

```python
try:
    x = int(input("Enter a number: "))
except ValueError as e:
    print("Invalid number!", e)
else:
    print("You entered:", x)
finally:
    print("This always runs.")
```

### 7.2 Raising Exceptions

```python
def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("b must not be zero")
    return a / b

print(divide(10, 2))
```

### 7.3 Custom Exceptions

```python
class ConfigurationError(Exception):
    """Raised when configuration is invalid."""


def load_config(path):
    if not path.endswith(".json"):
        raise ConfigurationError("Config file must be .json")
    # load file ...
```

**Tip:** Derive from `Exception` (or a subclass), not `BaseException`.

---

## 8. File Operations

### 8.1 Opening and Closing Files

```python
# Manual open/close (not recommended)
f = open("example.txt", mode="w", encoding="utf-8")
try:
    f.write("Hello\n")
finally:
    f.close()
```

### 8.2 Using Context Managers (Preferred)

```python
# 'with' automatically closes the file
with open("example.txt", mode="w", encoding="utf-8") as f:
    f.write("Hello, file!\n")
```

### 8.3 Reading Files

```python
with open("example.txt", mode="r", encoding="utf-8") as f:
    content = f.read()        # entire file as string

with open("example.txt", "r", encoding="utf-8") as f:
    first_line = f.readline()

with open("example.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()     # list of lines

# Iterating line by line (memory-efficient)
with open("example.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(line.strip())
```

### 8.4 Writing and Appending

```python
# Overwrite file
with open("log.txt", "w", encoding="utf-8") as f:
    f.write("Start log\n")

# Append to file
with open("log.txt", "a", encoding="utf-8") as f:
    f.write("Another entry\n")
```

---

## 9. Comprehensions

### 9.1 List Comprehensions

```python
nums = [1, 2, 3, 4, 5]

squares = [n * n for n in nums]

# With condition (filter)
evens = [n for n in nums if n % 2 == 0]

print(squares)  # [1, 4, 9, 16, 25]
print(evens)    # [2, 4]
```

### 9.2 Dictionary Comprehensions

```python
names = ["Alice", "Bob", "Charlie"]
name_lengths = {name: len(name) for name in names}
print(name_lengths)
```

### 9.3 Set Comprehensions

```python
words = ["apple", "banana", "apple", "cherry"]
unique_lengths = {len(word) for word in words}
print(unique_lengths)  # e.g. {5, 6}
```

### 9.4 Generator Expressions

```python
# Parentheses instead of []: produces a generator
nums = [1, 2, 3, 4, 5]

squares_gen = (n * n for n in nums)

for s in squares_gen:
    print(s)

# Common with sum()
total = sum(n * n for n in nums)
```

---

## 10. Important Built-in Functions

### 10.1 map, filter, reduce

```python
from functools import reduce

nums = [1, 2, 3, 4]

# map: apply function to each item
doubled = list(map(lambda x: x * 2, nums))

# filter: keep items where function returns True
evens = list(filter(lambda x: x % 2 == 0, nums))

# reduce: accumulate to a single value
product = reduce(lambda a, b: a * b, nums)

print(doubled)  # [2, 4, 6, 8]
print(evens)    # [2, 4]
print(product)  # 24

# Often clearer with comprehensions:
doubled2 = [x * 2 for x in nums]
```

### 10.2 zip, enumerate

```python
names = ["Alice", "Bob"]
ages = [25, 30]

for name, age in zip(names, ages):
    print(name, age)

for index, name in enumerate(names, start=1):
    print(index, name)
```

### 10.3 range, len, sum

```python
print(list(range(5)))          # [0, 1, 2, 3, 4]
print(list(range(1, 5)))       # [1, 2, 3, 4]
print(list(range(0, 10, 2)))   # [0, 2, 4, 6, 8]

nums = [1, 2, 3]
print(len(nums))  # 3
print(sum(nums))  # 6
```

### 10.4 sorted, reversed

```python
nums = [3, 1, 4, 1, 5]
print(sorted(nums))              # [1, 1, 3, 4, 5]
print(sorted(nums, reverse=True))

# Custom sort key
words = ["banana", "apple", "cherry"]
print(sorted(words, key=len))    # by length

# reversed returns an iterator
for n in reversed(nums):
    print(n)
```

### 10.5 any, all

```python
values = [0, 1, 2]
print(any(values))  # True (at least one truthy)
print(all(values))  # False (0 is falsy)

conditions = [x > 0 for x in [1, 2, 3]]
print(all(conditions))  # True
```

---

## 11. Modules and Packages

### 11.1 Importing Modules

```python
import math

print(math.sqrt(16))

import math as m
print(m.pi)
```

### 11.2 from ... import

```python
from math import sqrt, pi

print(sqrt(25))
print(pi)
```

**Gotcha:** Avoid `from module import *` in real code; it pollutes the namespace.

### 11.3 Creating Modules

Create a file `mymath.py`:

```python
# mymath.py

def add(a, b):
    return a + b
```

Use it from another file:

```python
import mymath

print(mymath.add(2, 3))
```

### 11.4 `if __name__ == "__main__"`

```python
# script.py

def main():
    print("Running as a script")

if __name__ == "__main__":
    main()
```

- When run directly: `__name__ == "__main__"` → runs `main()`.
- When imported: `__name__ == "script"` → `main()` not executed automatically.

---

## 12. Common Standard Library Modules

### 12.1 os and sys

```python
import os
import sys

print(os.getcwd())          # current working directory
print(os.listdir("."))     # files in current directory

print(sys.version)          # Python version
print(sys.argv)             # command-line arguments
```

### 12.2 datetime

```python
from datetime import datetime, date, timedelta

now = datetime.now()
print(now)

# Formatting
print(now.strftime("%Y-%m-%d %H:%M:%S"))

# Parsing
s = "2025-12-05"
parsed = datetime.strptime(s, "%Y-%m-%d")

# Date arithmetic
one_week = timedelta(weeks=1)
print(parsed + one_week)
```

### 12.3 math and random

```python
import math
import random

print(math.pi)
print(math.sqrt(16))

print(random.random())       # [0.0, 1.0)
print(random.randint(1, 6))  # inclusive
print(random.choice(["red", "green", "blue"]))
```

### 12.4 json

```python
import json

data = {"name": "Ada", "age": 36}

# Python -> JSON string
json_str = json.dumps(data)

# JSON string -> Python
loaded = json.loads(json_str)

# File operations
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f)

with open("data.json", "r", encoding="utf-8") as f:
    data2 = json.load(f)
```

### 12.5 csv

```python
import csv

# Writing
rows = [
    ["name", "age"],
    ["Ada", 36],
    ["Bob", 25],
]

with open("people.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerows(rows)

# Reading
with open("people.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

### 12.6 collections

```python
from collections import Counter, defaultdict, namedtuple, deque

# Counter
counts = Counter("banana")
print(counts)   # Counter({'a': 3, 'n': 2, 'b': 1})

# defaultdict
ages = defaultdict(int)
ages["Alice"] += 1
ages["Bob"] += 2
print(ages)

# namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(3, 4)
print(p.x, p.y)

# deque (fast appends/pops from both ends)
d = deque([1, 2, 3])
d.appendleft(0)
print(d)
```

### 12.7 itertools (Basics)

```python
import itertools as it

nums = [1, 2, 3]

print(list(it.product(nums, repeat=2)))   # Cartesian product
print(list(it.permutations(nums, 2)))     # permutations
print(list(it.combinations(nums, 2)))     # combinations

# Infinite iterators (be careful!)
counter = it.count(start=10, step=2)
for _ in range(3):
    print(next(counter))  # 10, 12, 14
```

---

## 13. Advanced Concepts

### 13.1 Generators and `yield`

```python
def countdown(n):
    print("Starting countdown")
    while n > 0:
        yield n
        n -= 1

for number in countdown(3):
    print(number)
```

- Generators produce values lazily, saving memory for large sequences.

### 13.2 Context Managers

#### Using `contextlib.contextmanager`

```python
from contextlib import contextmanager

@contextmanager
def open_file(path, mode):
    f = open(path, mode, encoding="utf-8")
    try:
        yield f
    finally:
        f.close()

with open_file("example.txt", "w") as f:
    f.write("Hello from custom context manager\n")
```

#### Implementing `__enter__` and `__exit__`

```python
class ManagedFile:
    def __init__(self, path, mode):
        self.path = path
        self.mode = mode
        self.file = None

    def __enter__(self):
        self.file = open(self.path, self.mode, encoding="utf-8")
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        # Return False to propagate exceptions
        return False

with ManagedFile("example2.txt", "w") as f:
    f.write("Managed file example\n")
```

### 13.3 Iterators and Iterables

- **Iterable**: can be looped over (`for ... in ...`). Has `__iter__`.
- **Iterator**: returned by `iter(iterable)`. Has `__next__`.

```python
nums = [1, 2, 3]           # iterable
it = iter(nums)           # iterator
print(next(it))           # 1
print(next(it))           # 2
print(next(it))           # 3

# Custom iterator
class CountDown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value

for n in CountDown(3):
    print(n)
```

### 13.4 Regular Expressions (re)

```python
import re

text = "Email me at test@example.com or admin@example.org"

# Find all email-like patterns
pattern = r"[\w.+-]+@[\w-]+\.[\w.-]+"
emails = re.findall(pattern, text)
print(emails)

# Search
m = re.search(r"(test)@example\.com", text)
if m:
    print(m.group(0))  # whole match
    print(m.group(1))  # first group: 'test'

# Substitution
redacted = re.sub(pattern, "<hidden>", text)
print(redacted)
```

**Tip:** Use raw strings (`r"..."`) for regex patterns to avoid escaping backslashes.

---

## 14. Type Hints (Typing)

```python
from typing import Optional, Iterable


def greet(name: str, times: int = 1) -> str:
    return "\n".join([f"Hello, {name}!" for _ in range(times)])


def total(values: Iterable[int]) -> int:
    return sum(values)

age: Optional[int] = None
age = 30
```

- Use `list[int]`, `dict[str, int]` (Python 3.9+).
- Optional type: `Optional[int]` or `int | None` (3.10+).
- Type hints are not enforced at runtime; use tools like `mypy`, `pyright`.

---

## 15. Async Basics (`asyncio`)

```python
import asyncio

async def task(name, delay):
    await asyncio.sleep(delay)
    print(f"Task {name} finished after {delay}s")

async def main():
    await asyncio.gather(
        task("A", 1),
        task("B", 2),
    )

if __name__ == "__main__":
    asyncio.run(main())
```

- `async def` defines a coroutine.
- `await` suspends execution until the awaited coroutine/future is done.
- `asyncio.gather` runs coroutines concurrently.

---

### Final Tips

- Prefer readability over clever one-liners.
- Use virtual environments (`python -m venv .venv`) and `pip` to manage dependencies.
- Run `python -m this` to read the "Zen of Python" for guiding principles.
