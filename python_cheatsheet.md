# Python Programming Cheatsheet (Comprehensive)

**Version:** Python 3.8+ | **Last Updated:** December 2025

A complete reference guide covering Python from basics to advanced concepts with practical examples and best practices.

---

## 1. BASICS

### 1.1 Variables & Data Types

```python
# Integer
age = 25
print(type(age))  # <class 'int'>

# Float
price = 19.99
print(type(price))  # <class 'float'>

# String
name = "Alice"
print(type(name))  # <class 'str'>

# Boolean
is_active = True
print(type(is_active))  # <class 'bool'>

# None (null value)
result = None
print(type(result))  # <class 'NoneType'>

# Complex numbers
complex_num = 3 + 4j
print(type(complex_num))  # <class 'complex'>
print(complex_num.real)   # 3.0
print(complex_num.imag)   # 4.0
```

### 1.2 Type System

```python
# type() - get type of object
x = 42
print(type(x))  # <class 'int'>

# isinstance() - check if object is instance of class
print(isinstance(x, int))        # True
print(isinstance(x, (int, str))) # True (checks multiple types)

# Type conversion
x = "123"
y = int(x)       # String to int
z = float(x)     # String to float
w = str(456)     # Int to string
print(y, z, w)   # 123 123.0 456

# Duck typing - if it walks like a duck and quacks like a duck...
# Python doesn't care about type, only that object has required methods
def process(data):
    return data.upper()  # Works with any object that has .upper()

print(process("hello"))  # HELLO
```

### 1.3 Truthiness (Falsy Values)

```python
# Complete list of falsy values in Python
falsy_values = [
    False,      # Boolean False
    None,       # None type
    0,          # Zero integer
    0.0,        # Zero float
    0j,         # Zero complex
    "",         # Empty string
    [],         # Empty list
    {},         # Empty dictionary
    (),         # Empty tuple
    set(),      # Empty set
    frozenset() # Empty frozenset
]

for val in falsy_values:
    if not val:
        print(f"{repr(val)} is falsy")

# Everything else is truthy
truthy = [1, "0", [0], " ", True]
for val in truthy:
    if val:
        print(f"{repr(val)} is truthy")
```

### 1.4 Numeric Literals

```python
# Decimal (base 10)
decimal = 1000
print(decimal)  # 1000

# Hexadecimal (base 16) - prefix with 0x
hexadecimal = 0xFF
print(hexadecimal)  # 255

# Binary (base 2) - prefix with 0b
binary = 0b1010
print(binary)  # 10

# Octal (base 8) - prefix with 0o
octal = 0o17
print(octal)  # 15

# Underscore separators for readability (Python 3.6+)
million = 1_000_000
print(million)  # 1000000

# Works with hex, binary, octal too
hex_value = 0xFF_FF_FF
print(hex_value)  # 16777215
```

### 1.5 Multiple Assignment

```python
# Simultaneous assignment
a, b = 1, 2
print(a, b)  # 1 2

# Chain assignment
a = b = c = 5
print(a, b, c)  # 5 5 5

# Swapping variables (Python's elegant way)
x, y = 10, 20
x, y = y, x
print(x, y)  # 20 10

# Extended unpacking (Python 3.0+)
first, *middle, last = [1, 2, 3, 4, 5]
print(first)   # 1
print(middle)  # [2, 3, 4]
print(last)    # 5

# Ignoring values
a, _, c = (1, 2, 3)
print(a, c)  # 1 3
```

### 1.6 Comments & Docstrings

```python
# Single-line comment

"""
Multi-line comment
Can span multiple lines
Using triple quotes
"""

def function_example(param1, param2):
    """
    Short one-line description of function.
    
    More detailed description if needed. Explain what the function
    does, its parameters, return value, and any exceptions.
    
    Args:
        param1 (int): Description of param1
        param2 (str): Description of param2
    
    Returns:
        bool: Description of return value
    
    Raises:
        ValueError: When input is invalid
    
    Example:
        >>> function_example(5, "test")
        True
    """
    pass

# PEP 257 conventions:
# - Use triple double-quotes """
# - One-line docstrings on same line as quotes
# - Multi-line docstrings have summary on first line
# - Blank line before closing quotes in multi-line
```

### 1.7 Input/Output

```python
# Basic input (always returns string)
name = input("Enter your name: ")
print(f"Hello, {name}!")

# Convert input to int
age = int(input("Enter age: "))

# print() with parameters
print("Hello", "World")              # Hello World (default sep=' ')
print("Hello", "World", sep=", ")    # Hello, World
print("Hello", end=" ")
print("World")                        # Hello World (end parameter)

# Print to file
with open("output.txt", "w") as f:
    print("Hello, file!", file=f)

# Multiple values
x, y, z = 1, 2, 3
print(x, y, z)                    # 1 2 3
print(x, y, z, sep=", ")          # 1, 2, 3
print(x, y, z, sep="\n")          # Each on new line

# Formatted output (covered more in Strings section)
name, age = "Alice", 30
print(f"{name} is {age} years old")  # Alice is 30 years old
```

### 1.8 String Representations

```python
# str() vs repr()
# str() - human-readable string
# repr() - unambiguous representation (for developers)

class Person:
    def __init__(self, name):
        self.name = name
    
    def __str__(self):
        return f"Person named {self.name}"
    
    def __repr__(self):
        return f"Person('{self.name}')"

p = Person("Alice")
print(str(p))   # Person named Alice
print(repr(p))  # Person('Alice')

# For built-in types
s = "Hello\nWorld"
print(str(s))   # Hello (with newline) World
print(repr(s))  # 'Hello\nWorld' (shows escape sequence)

# ASCII vs Unicode
text = "Hello, 世界"
print(text.encode('utf-8'))    # b'Hello, \xe4\xb8\x96\xe7\x95\x8c'
print(text.encode('ascii', errors='ignore'))  # b'Hello, '
```

---

## 2. OPERATORS

### 2.1 Arithmetic Operators

```python
# Basic arithmetic
a, b = 10, 3

print(a + b)   # 13 (addition)
print(a - b)   # 7 (subtraction)
print(a * b)   # 30 (multiplication)
print(a / b)   # 3.333... (true division, always returns float)
print(a // b)  # 3 (floor division, rounds down)
print(a % b)   # 1 (modulo, remainder)
print(a ** b)  # 1000 (exponentiation)

# Operator precedence (highest to lowest)
result = 2 + 3 * 4 ** 2  # Exponentiation first, then multiplication, then addition
print(result)  # 50 (not 80 or 400)

# Use parentheses for clarity
result = (2 + 3) * 4 ** 2
print(result)  # 80
```

### 2.2 Comparison Operators

```python
a, b = 5, 10

print(a == b)  # False (equal to)
print(a != b)  # True (not equal to)
print(a < b)   # True (less than)
print(a > b)   # False (greater than)
print(a <= b)  # True (less than or equal)
print(a >= b)  # False (greater than or equal)

# Chained comparisons (Pythonic!)
x = 5
print(1 < x < 10)        # True (equivalent to: 1 < x and x < 10)
print(1 < x < 3)         # False
print(10 > x >= 5)       # True

# Can chain multiple comparisons
print(1 < 2 < 3 < 4)     # True
```

### 2.3 Logical Operators

```python
# and, or, not
a, b = True, False

print(a and b)  # False
print(a or b)   # True
print(not a)    # False

# Short-circuit evaluation
# 'and' returns first falsy value or last value
print(0 and 5)      # 0 (first falsy)
print(5 and 3)      # 3 (both truthy, returns last)
print(5 and 0)      # 0 (second is falsy)

# 'or' returns first truthy value or last value
print(0 or 5)       # 5 (first falsy, returns second)
print(5 or 3)       # 5 (first truthy)
print(0 or False)   # False (both falsy, returns last)

# Practical use of short-circuit
def expensive_function():
    print("Expensive function called!")
    return True

flag = False
result = flag and expensive_function()  # expensive_function() not called!
print(result)  # False
```

### 2.4 Bitwise Operators

```python
a, b = 60, 13  # 60 = 0011 1100, 13 = 0000 1101

print(a & b)   # 12 (0000 1100) - AND
print(a | b)   # 61 (0011 1101) - OR
print(a ^ b)   # 49 (0011 0001) - XOR
print(~a)      # -61 (inverts all bits)
print(a << 2)  # 240 (shift left by 2)
print(a >> 2)  # 15 (shift right by 2)

# Practical uses
# Check if number is even
print(10 & 1 == 0)  # True (even)
print(11 & 1 == 0)  # False (odd)

# Set specific bit
flags = 0b0000
flags |= 0b0100  # Set bit 2
print(bin(flags))  # 0b100

# Clear specific bit
flags &= ~0b0100
print(bin(flags))  # 0b0
```

### 2.5 Assignment Operators

```python
x = 10

# Augmented assignment operators
x += 5   # x = x + 5
print(x)  # 15

x -= 3   # x = x - 3
print(x)  # 12

x *= 2   # x = x * 2
print(x)  # 24

x /= 4   # x = x / 4
print(x)  # 6.0

x //= 2  # x = x // 2
print(x)  # 3.0

x %= 2   # x = x % 2
print(x)  # 1.0

x **= 3  # x = x ** 3
print(x)  # 1.0

# Bitwise augmented assignment
y = 8
y &= 3   # y = y & 3
y |= 4   # y = y | 4
y ^= 2   # y = y ^ 2
y >>= 1  # y = y >> 1
y <<= 1  # y = y << 1
```

### 2.6 Identity Operators

```python
# 'is' checks if two references point to same object
# '==' checks if two objects have same value

a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)    # True (same values)
print(a is b)    # False (different objects)
print(a is c)    # True (same object)

# When to use 'is':
# 1. Comparing with None
x = None
print(x is None)  # Correct way

# 2. Comparing with True/False
flag = True
print(flag is True)  # Works, but 'flag' or 'not flag' is more Pythonic

# 3. Checking object identity
if some_var is not None:
    print("Variable is not None")

# Integer caching: Python caches small integers (-5 to 256)
a = 256
b = 256
print(a is b)  # True (cached)

a = 257
b = 257
print(a is b)  # False (not cached, different objects)
```

### 2.7 Membership Operators

```python
# 'in' and 'not in' - check if value exists in sequence

# Lists
numbers = [1, 2, 3, 4, 5]
print(3 in numbers)      # True
print(6 not in numbers)  # True

# Strings
text = "Hello, World!"
print("World" in text)   # True
print("world" in text)   # False (case-sensitive)

# Dictionaries (checks keys)
person = {"name": "Alice", "age": 30}
print("name" in person)   # True
print("Alice" in person)  # False (values not checked)
print("Alice" in person.values())  # True

# Sets
colors = {"red", "green", "blue"}
print("red" in colors)    # True
```

### 2.8 Walrus Operator (Python 3.8+)

```python
# := assigns and returns value in single expression

# Without walrus operator
data = input("Enter something: ")
if len(data) > 5:
    print(f"Input too long: {len(data)} characters")

# With walrus operator (more concise)
if (n := len(input("Enter something: "))) > 5:
    print(f"Input too long: {n} characters")

# Useful in while loops
# Without walrus
line = input("Enter line: ")
while line != "quit":
    print(f"You entered: {line}")
    line = input("Enter line: ")

# With walrus
while (line := input("Enter line: ")) != "quit":
    print(f"You entered: {line}")

# List comprehensions
# Get squares of numbers that are > 5 after squaring
numbers = [1, 2, 3, 4, 5]
squares = [square for x in numbers if (square := x**2) > 5]
print(squares)  # [9, 16, 25]
```

### 2.9 Operator Precedence Table

```python
# From highest to lowest precedence:
"""
1. ()                           - Parentheses
2. **                           - Exponentiation
3. +x, -x, ~x                   - Unary plus, minus, bitwise NOT
4. *, /, //, %                  - Multiplication, division, floor div, modulo
5. +, -                         - Addition, subtraction
6. <<, >>                       - Bitwise shifts
7. &                            - Bitwise AND
8. ^                            - Bitwise XOR
9. |                            - Bitwise OR
10. ==, !=, <, >, <=, >=,      - Comparisons
    is, is not, in, not in
11. not                         - Boolean NOT
12. and                         - Boolean AND
13. or                          - Boolean OR
14. :=                          - Walrus operator
"""

# Example demonstrating precedence
result = 2 + 3 * 4 ** 2 / 8 - 1
# 4 ** 2 = 16
# 3 * 16 = 48
# 48 / 8 = 6.0
# 2 + 6.0 = 8.0
# 8.0 - 1 = 7.0
print(result)  # 7.0
```

---

## 3. DATA STRUCTURES

### 3.1 Lists

#### 3.1.1 Creation Methods

```python
# Empty list
empty = []
empty2 = list()

# List with elements
numbers = [1, 2, 3, 4, 5]
mixed = [1, "two", 3.0, [4, 5]]  # Can contain different types

# List from iterable
chars = list("hello")
print(chars)  # ['h', 'e', 'l', 'l', 'o']

# List comprehension (covered later in detail)
squares = [x**2 for x in range(5)]
print(squares)  # [0, 1, 4, 9, 16]

# list() constructor
numbers = list(range(1, 6))
print(numbers)  # [1, 2, 3, 4, 5]
```

#### 3.1.2 Indexing & Slicing

```python
fruits = ['apple', 'banana', 'cherry', 'date', 'elderberry']

# Positive indexing (0-based)
print(fruits[0])   # 'apple'
print(fruits[2])   # 'cherry'

# Negative indexing (from end)
print(fruits[-1])  # 'elderberry'
print(fruits[-2])  # 'date'

# IndexError if out of bounds
# print(fruits[10])  # Raises IndexError

# Slicing: list[start:stop:step]
print(fruits[1:4])      # ['banana', 'cherry', 'date']
print(fruits[:3])       # ['apple', 'banana', 'cherry'] (from start)
print(fruits[2:])       # ['cherry', 'date', 'elderberry'] (to end)
print(fruits[::2])      # ['apple', 'cherry', 'elderberry'] (every 2nd)
print(fruits[::-1])     # Reverse list

# Slice object
s = slice(1, 4)
print(fruits[s])  # ['banana', 'cherry', 'date']

# Slicing never raises IndexError
print(fruits[10:20])  # [] (empty list)
```

#### 3.1.3 List Methods

```python
fruits = ['apple', 'banana']

# append() - add single element at end
fruits.append('cherry')
print(fruits)  # ['apple', 'banana', 'cherry']

# extend() - add multiple elements
fruits.extend(['date', 'elderberry'])
print(fruits)  # ['apple', 'banana', 'cherry', 'date', 'elderberry']

# insert() - add at specific position
fruits.insert(1, 'apricot')
print(fruits)  # ['apple', 'apricot', 'banana', 'cherry', 'date', 'elderberry']

# remove() - remove first occurrence
fruits.remove('banana')
print(fruits)  # ['apple', 'apricot', 'cherry', 'date', 'elderberry']

# pop() - remove and return element (default: last)
last = fruits.pop()
print(last)    # 'elderberry'
second = fruits.pop(1)
print(second)  # 'apricot'
print(fruits)  # ['apple', 'cherry', 'date']

# clear() - remove all elements
temp = [1, 2, 3]
temp.clear()
print(temp)  # []

# index() - find index of element
fruits = ['apple', 'banana', 'cherry']
print(fruits.index('banana'))  # 1
# print(fruits.index('date'))  # Raises ValueError

# count() - count occurrences
numbers = [1, 2, 2, 3, 2, 4]
print(numbers.count(2))  # 3

# reverse() - reverse in place
numbers.reverse()
print(numbers)  # [4, 2, 3, 2, 2, 1]

# sort() - sort in place
numbers.sort()
print(numbers)  # [1, 2, 2, 2, 3, 4]
numbers.sort(reverse=True)
print(numbers)  # [4, 3, 2, 2, 2, 1]

# copy() - shallow copy
original = [1, 2, [3, 4]]
copied = original.copy()
copied[0] = 999
copied[2][0] = 999
print(original)  # [1, 2, [999, 4]] - nested list affected!
print(copied)    # [999, 2, [999, 4]]
```

#### 3.1.4 List Operations

```python
# Concatenation
list1 = [1, 2, 3]
list2 = [4, 5, 6]
combined = list1 + list2
print(combined)  # [1, 2, 3, 4, 5, 6]

# Repetition
repeated = [1, 2] * 3
print(repeated)  # [1, 2, 1, 2, 1, 2]

# Membership
print(2 in [1, 2, 3])  # True
print(5 not in [1, 2, 3])  # True

# Length
print(len([1, 2, 3, 4]))  # 4

# Min/Max (for comparable elements)
numbers = [3, 1, 4, 1, 5]
print(min(numbers))  # 1
print(max(numbers))  # 5
print(sum(numbers))  # 14
```

#### 3.1.5 List Comprehensions

```python
# Basic list comprehension
squares = [x**2 for x in range(10)]
print(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# With condition
evens = [x for x in range(10) if x % 2 == 0]
print(evens)  # [0, 2, 4, 6, 8]

# With if-else
labels = ["even" if x % 2 == 0 else "odd" for x in range(5)]
print(labels)  # ['even', 'odd', 'even', 'odd', 'even']

# Nested comprehensions
matrix = [[i*j for j in range(3)] for i in range(3)]
print(matrix)  # [[0, 0, 0], [0, 1, 2], [0, 2, 4]]

# Multiple for clauses
pairs = [(x, y) for x in range(3) for y in range(3)]
print(pairs)  # [(0,0), (0,1), (0,2), (1,0), (1,1), (1,2), (2,0), (2,1), (2,2)]

# Flattening nested list
nested = [[1, 2], [3, 4], [5, 6]]
flat = [item for sublist in nested for item in sublist]
print(flat)  # [1, 2, 3, 4, 5, 6]
```

#### 3.1.6 Common Gotchas

```python
# GOTCHA 1: Shallow vs Deep copy
import copy

original = [[1, 2], [3, 4]]
shallow = original.copy()
deep = copy.deepcopy(original)

original[0][0] = 999
print(original)  # [[999, 2], [3, 4]]
print(shallow)   # [[999, 2], [3, 4]] - affected!
print(deep)      # [[1, 2], [3, 4]] - not affected

# GOTCHA 2: Mutable default arguments
def append_to(element, target=[]):  # DON'T DO THIS!
    target.append(element)
    return target

print(append_to(1))  # [1]
print(append_to(2))  # [1, 2] - unexpected!

# FIX: Use None as default
def append_to_fixed(element, target=None):
    if target is None:
        target = []
    target.append(element)
    return target

print(append_to_fixed(1))  # [1]
print(append_to_fixed(2))  # [2] - correct!

# GOTCHA 3: List repetition with mutable objects
matrix = [[0] * 3] * 3  # DON'T DO THIS!
matrix[0][0] = 1
print(matrix)  # [[1, 0, 0], [1, 0, 0], [1, 0, 0]] - all rows affected!

# FIX: Use list comprehension
matrix = [[0] * 3 for _ in range(3)]
matrix[0][0] = 1
print(matrix)  # [[1, 0, 0], [0, 0, 0], [0, 0, 0]] - correct!
```

### 3.2 Tuples

#### 3.2.1 Creation & Immutability

```python
# Empty tuple
empty = ()
empty2 = tuple()

# Single element tuple (note the comma!)
single = (1,)  # Comma is required
not_tuple = (1)  # This is just int 1 with parentheses
print(type(single))    # <class 'tuple'>
print(type(not_tuple)) # <class 'int'>

# Multiple elements
coordinates = (10, 20)
rgb = (255, 128, 0)

# Without parentheses (tuple packing)
point = 10, 20, 30
print(type(point))  # <class 'tuple'>

# tuple() constructor
numbers = tuple([1, 2, 3])
chars = tuple("hello")
print(chars)  # ('h', 'e', 'l', 'l', 'o')

# Immutability
try:
    coordinates[0] = 5  # TypeError!
except TypeError as e:
    print(f"Error: {e}")

# But tuples can contain mutable objects
mixed = (1, 2, [3, 4])
mixed[2][0] = 999  # This works!
print(mixed)  # (1, 2, [999, 4])
```

#### 3.2.2 Tuple Methods & Operations

```python
numbers = (1, 2, 2, 3, 2, 4)

# count() - count occurrences
print(numbers.count(2))  # 3

# index() - find first occurrence
print(numbers.index(3))  # 3
# print(numbers.index(5))  # ValueError

# Length
print(len(numbers))  # 6

# Min/Max/Sum
print(min(numbers))  # 1
print(max(numbers))  # 4
print(sum(numbers))  # 14

# Membership
print(2 in numbers)  # True

# Concatenation
tuple1 = (1, 2, 3)
tuple2 = (4, 5, 6)
combined = tuple1 + tuple2
print(combined)  # (1, 2, 3, 4, 5, 6)

# Repetition
repeated = (1, 2) * 3
print(repeated)  # (1, 2, 1, 2, 1, 2)

# Indexing and slicing (same as lists)
print(numbers[0])    # 1
print(numbers[-1])   # 4
print(numbers[1:4])  # (2, 2, 3)
```

#### 3.2.3 Packing & Unpacking

```python
# Tuple packing
person = "Alice", 30, "Engineer"

# Tuple unpacking
name, age, job = person
print(name, age, job)  # Alice 30 Engineer

# Extended unpacking (Python 3.0+)
numbers = (1, 2, 3, 4, 5)
first, *middle, last = numbers
print(first)   # 1
print(middle)  # [2, 3, 4] (becomes list!)
print(last)    # 5

# Swapping
a, b = 10, 20
a, b = b, a
print(a, b)  # 20 10

# Ignore values
x, _, z = (1, 2, 3)
print(x, z)  # 1 3

# Function returning multiple values (uses tuple)
def get_user():
    return "Alice", 30, "alice@example.com"

name, age, email = get_user()
print(name, age, email)  # Alice 30 alice@example.com
```

#### 3.2.4 Named Tuples

```python
from collections import namedtuple

# Define named tuple
Point = namedtuple('Point', ['x', 'y'])
Person = namedtuple('Person', 'name age job')  # Can use string

# Create instances
p = Point(10, 20)
alice = Person("Alice", 30, "Engineer")

# Access by name or index
print(p.x, p.y)           # 10 20
print(p[0], p[1])         # 10 20
print(alice.name)         # Alice
print(alice[0])           # Alice

# Immutable
# p.x = 5  # AttributeError

# Convert to dict
print(alice._asdict())  # {'name': 'Alice', 'age': 30, 'job': 'Engineer'}

# Replace (creates new tuple)
bob = alice._replace(name="Bob")
print(bob)  # Person(name='Bob', age=30, job='Engineer')

# Useful for readable code
def get_coordinates():
    return Point(100, 200)

point = get_coordinates()
print(f"X: {point.x}, Y: {point.y}")  # More readable than point[0], point[1]
```

#### 3.2.5 Use Cases for Tuples

```python
# 1. Multiple return values
def divide_with_remainder(a, b):
    return a // b, a % b

quotient, remainder = divide_with_remainder(17, 5)
print(quotient, remainder)  # 3 2

# 2. Dictionary keys (tuples are hashable, lists are not)
locations = {
    (0, 0): "Origin",
    (1, 0): "Right",
    (0, 1): "Up"
}
print(locations[(0, 0)])  # Origin

# 3. Immutable records
student = ("Alice", 20, "CS")
# Protects data from accidental modification

# 4. Function arguments
def print_point(x, y, z):
    print(f"Point: ({x}, {y}, {z})")

point = (1, 2, 3)
print_point(*point)  # Unpacking tuple as arguments

# 5. Guaranteed immutability
config = ("localhost", 8080, False)
# Can't accidentally change configuration
```

### 3.3 Sets

#### 3.3.1 Creation

```python
# Empty set (must use set(), not {})
empty = set()  # Correct
# empty = {}  # This creates empty dict!

# Set with elements
numbers = {1, 2, 3, 4, 5}
print(numbers)  # {1, 2, 3, 4, 5} (order not guaranteed)

# Sets automatically remove duplicates
duplicates = {1, 2, 2, 3, 3, 3}
print(duplicates)  # {1, 2, 3}

# set() constructor
from_list = set([1, 2, 2, 3])
from_string = set("hello")
print(from_list)    # {1, 2, 3}
print(from_string)  # {'h', 'e', 'l', 'o'}

# Sets can only contain hashable (immutable) elements
# valid = {1, "two", (3, 4)}  # OK
# invalid = {[1, 2]}  # TypeError: unhashable type: 'list'

# frozenset - immutable set
frozen = frozenset([1, 2, 3])
# frozen.add(4)  # AttributeError
```

#### 3.3.2 Set Operations

```python
set1 = {1, 2, 3, 4}
set2 = {3, 4, 5, 6}

# Union (all elements from both) - |
print(set1 | set2)  # {1, 2, 3, 4, 5, 6}
print(set1.union(set2))

# Intersection (common elements) - &
print(set1 & set2)  # {3, 4}
print(set1.intersection(set2))

# Difference (in first but not second) - -
print(set1 - set2)  # {1, 2}
print(set1.difference(set2))

# Symmetric difference (in either but not both) - ^
print(set1 ^ set2)  # {1, 2, 5, 6}
print(set1.symmetric_difference(set2))

# Subset and superset
set3 = {1, 2}
print(set3 <= set1)  # True (is subset)
print(set3.issubset(set1))
print(set1 >= set3)  # True (is superset)
print(set1.issuperset(set3))

# Disjoint (no common elements)
set4 = {7, 8, 9}
print(set1.isdisjoint(set4))  # True
```

#### 3.3.3 Set Methods

```python
fruits = {"apple", "banana"}

# add() - add single element
fruits.add("cherry")
print(fruits)  # {'apple', 'banana', 'cherry'}

# update() - add multiple elements
fruits.update(["date", "elderberry"])
fruits.update("fg")  # Adds 'f' and 'g'
print(fruits)

# remove() - remove element (raises KeyError if not found)
fruits.remove("banana")
# fruits.remove("xyz")  # KeyError!

# discard() - remove element (no error if not found)
fruits.discard("apple")
fruits.discard("xyz")  # No error

# pop() - remove and return arbitrary element
item = fruits.pop()
print(f"Removed: {item}")

# clear() - remove all elements
temp = {1, 2, 3}
temp.clear()
print(temp)  # set()

# copy() - shallow copy
original = {1, 2, 3}
copied = original.copy()

# In-place operations
set1 = {1, 2, 3}
set2 = {3, 4, 5}
set1 |= set2  # Union in place
print(set1)  # {1, 2, 3, 4, 5}

set1 &= {1, 2, 4}  # Intersection in place
print(set1)  # {1, 2, 4}
```

#### 3.3.4 Set Comprehensions

```python
# Basic set comprehension
squares = {x**2 for x in range(10)}
print(squares)  # {0, 1, 4, 9, 16, 25, 36, 49, 64, 81}

# With condition
even_squares = {x**2 for x in range(10) if x % 2 == 0}
print(even_squares)  # {0, 4, 16, 36, 64}

# Remove duplicates from list
numbers = [1, 2, 2, 3, 3, 3, 4]
unique = {x for x in numbers}
print(unique)  # {1, 2, 3, 4}
```

#### 3.3.5 Practical Use Cases

```python
# Remove duplicates
numbers = [1, 2, 2, 3, 3, 3]
unique = list(set(numbers))
print(unique)  # [1, 2, 3] (order not preserved)

# Membership testing (O(1) vs O(n) for lists)
large_set = set(range(1000000))
large_list = list(range(1000000))
# 999999 in large_set  # Very fast
# 999999 in large_list  # Much slower

# Find common elements
list1 = [1, 2, 3, 4, 5]
list2 = [4, 5, 6, 7, 8]
common = set(list1) & set(list2)
print(common)  # {4, 5}

# Find unique elements
all_unique = set(list1) ^ set(list2)
print(all_unique)  # {1, 2, 3, 6, 7, 8}
```

### 3.4 Dictionaries

#### 3.4.1 Creation

```python
# Empty dictionary
empty = {}
empty2 = dict()

# Dictionary with key-value pairs
person = {
    "name": "Alice",
    "age": 30,
    "job": "Engineer"
}

# dict() constructor
person2 = dict(name="Bob", age=25, job="Designer")
print(person2)  # {'name': 'Bob', 'age': 25, 'job': 'Designer'}

# From list of tuples
pairs = [("a", 1), ("b", 2), ("c", 3)]
d = dict(pairs)
print(d)  # {'a': 1, 'b': 2, 'c': 3}

# Dictionary comprehension
squares = {x: x**2 for x in range(5)}
print(squares)  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# fromkeys() - create dict with same value for all keys
keys = ['a', 'b', 'c']
default_dict = dict.fromkeys(keys, 0)
print(default_dict)  # {'a': 0, 'b': 0, 'c': 0}
```

#### 3.4.2 Accessing Elements

```python
person = {"name": "Alice", "age": 30, "job": "Engineer"}

# Using []
print(person["name"])  # Alice
# print(person["salary"])  # KeyError!

# Using get() - returns None if key doesn't exist
print(person.get("name"))     # Alice
print(person.get("salary"))   # None
print(person.get("salary", 0))  # 0 (custom default)

# setdefault() - get value or set default if key doesn't exist
salary = person.setdefault("salary", 50000)
print(salary)  # 50000
print(person)  # Now has 'salary' key

# Check if key exists
if "name" in person:
    print("Name exists")

# Check if key doesn't exist
if "address" not in person:
    print("Address not found")
```

#### 3.4.3 Dictionary Methods

```python
person = {"name": "Alice", "age": 30, "job": "Engineer"}

# keys() - get all keys
print(person.keys())  # dict_keys(['name', 'age', 'job'])
print(list(person.keys()))  # ['name', 'age', 'job']

# values() - get all values
print(person.values())  # dict_values(['Alice', 30, 'Engineer'])

# items() - get key-value pairs as tuples
print(person.items())  # dict_items([('name', 'Alice'), ('age', 30), ('job', 'Engineer')])

# Iteration
for key in person:
    print(key, person[key])

for key, value in person.items():
    print(f"{key}: {value}")

# pop() - remove and return value
age = person.pop("age")
print(age)     # 30
print(person)  # {'name': 'Alice', 'job': 'Engineer'}

# pop with default (no KeyError)
salary = person.pop("salary", 0)
print(salary)  # 0

# popitem() - remove and return last key-value pair (Python 3.7+)
item = person.popitem()
print(item)    # ('job', 'Engineer')

# update() - merge dictionaries
person.update({"age": 31, "city": "NYC"})
print(person)  # {'name': 'Alice', 'age': 31, 'city': 'NYC'}

# clear() - remove all items
temp = {"a": 1, "b": 2}
temp.clear()
print(temp)  # {}

# copy() - shallow copy
original = {"a": 1, "b": 2}
copied = original.copy()
```

#### 3.4.4 Dictionary Merging

```python
dict1 = {"a": 1, "b": 2}
dict2 = {"c": 3, "d": 4}
dict3 = {"b": 20, "e": 5}  # Note: 'b' exists in dict1

# Using ** unpacking
merged = {**dict1, **dict2}
print(merged)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

merged_overlap = {**dict1, **dict3}
print(merged_overlap)  # {'a': 1, 'b': 20, 'e': 5} - later value wins

# Using | operator (Python 3.9+)
merged2 = dict1 | dict2
print(merged2)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# In-place merge with |= (Python 3.9+)
dict1_copy = dict1.copy()
dict1_copy |= dict2
print(dict1_copy)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# Using update()
dict1_copy2 = dict1.copy()
dict1_copy2.update(dict2)
print(dict1_copy2)
```

#### 3.4.5 Dictionary Comprehensions

```python
# Basic dictionary comprehension
squares = {x: x**2 for x in range(5)}
print(squares)  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# With condition
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}
print(even_squares)  # {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}

# Swapping keys and values
original = {"a": 1, "b": 2, "c": 3}
swapped = {v: k for k, v in original.items()}
print(swapped)  # {1: 'a', 2: 'b', 3: 'c'}

# From two lists
keys = ['name', 'age', 'job']
values = ['Alice', 30, 'Engineer']
person = {k: v for k, v in zip(keys, values)}
print(person)  # {'name': 'Alice', 'age': 30, 'job': 'Engineer'}

# Filtering dictionary
scores = {'Alice': 85, 'Bob': 72, 'Charlie': 90, 'David': 65}
passed = {name: score for name, score in scores.items() if score >= 75}
print(passed)  # {'Alice': 85, 'Charlie': 90}
```

#### 3.4.6 Collections Module Dictionaries

```python
from collections import defaultdict, Counter, OrderedDict, ChainMap

# defaultdict - provides default value for missing keys
dd = defaultdict(int)  # default value: 0
dd['a'] += 1
dd['b'] += 2
print(dd)  # defaultdict(<class 'int'>, {'a': 1, 'b': 2})
print(dd['c'])  # 0 (no KeyError!)

# With list as default
dd_list = defaultdict(list)
dd_list['fruits'].append('apple')
dd_list['fruits'].append('banana')
print(dd_list)  # defaultdict(<class 'list'>, {'fruits': ['apple', 'banana']})

# Counter - count occurrences
words = ['apple', 'banana', 'apple', 'cherry', 'banana', 'apple']
counts = Counter(words)
print(counts)  # Counter({'apple': 3, 'banana': 2, 'cherry': 1})
print(counts['apple'])  # 3
print(counts.most_common(2))  # [('apple', 3), ('banana', 2)]

# Operations with Counter
c1 = Counter(['a', 'b', 'c', 'a'])
c2 = Counter(['a', 'b', 'd'])
print(c1 + c2)  # Counter({'a': 3, 'b': 2, 'c': 1, 'd': 1})
print(c1 - c2)  # Counter({'a': 1, 'c': 1})

# OrderedDict - maintains insertion order (less relevant in Python 3.7+)
# Regular dicts maintain order in 3.7+, but OrderedDict has additional methods
od = OrderedDict()
od['first'] = 1
od['second'] = 2
od['third'] = 3
print(od)

# move_to_end()
od.move_to_end('first')
print(od)  # 'first' is now at the end

# ChainMap - combine multiple dicts
dict1 = {'a': 1, 'b': 2}
dict2 = {'c': 3, 'd': 4}
dict3 = {'e': 5, 'a': 10}  # 'a' also in dict1
chain = ChainMap(dict1, dict2, dict3)
print(chain['a'])  # 1 (from first dict)
print(chain['c'])  # 3
print(list(chain.keys()))  # All keys from all dicts
```

#### 3.4.7 Dictionary Gotchas

```python
# GOTCHA 1: Mutable keys (not allowed)
# bad = {[1, 2]: "value"}  # TypeError: unhashable type: 'list'
# Use tuple instead:
good = {(1, 2): "value"}

# GOTCHA 2: KeyError handling
person = {"name": "Alice"}
# print(person["age"])  # KeyError!
# Better:
age = person.get("age", 25)  # Returns default if key doesn't exist

# GOTCHA 3: Iterating while modifying
d = {'a': 1, 'b': 2, 'c': 3}
# for key in d:
#     if key == 'b':
#         del d[key]  # RuntimeError: dictionary changed size during iteration

# FIX: Iterate over copy
for key in list(d.keys()):  # or list(d)
    if key == 'b':
        del d[key]
print(d)  # {'a': 1, 'c': 3}

# GOTCHA 4: Default mutable values in fromkeys()
keys = ['a', 'b', 'c']
d = dict.fromkeys(keys, [])  # DON'T DO THIS!
d['a'].append(1)
print(d)  # {'a': [1], 'b': [1], 'c': [1]} - all share same list!

# FIX: Use defaultdict or dict comprehension
d = {k: [] for k in keys}
d['a'].append(1)
print(d)  # {'a': [1], 'b': [], 'c': []} - correct!
```

### 3.5 Strings

#### 3.5.1 String Creation

```python
# Single quotes
s1 = 'Hello'

# Double quotes
s2 = "World"

# Triple quotes (multi-line)
s3 = """This is
a multi-line
string"""

s4 = '''Also
works with
single quotes'''

# Raw strings (escape sequences not processed)
path = r"C:\Users\name\folder"
print(path)  # C:\Users\name\folder (backslashes not escaped)

# f-strings (formatted string literals, Python 3.6+)
name, age = "Alice", 30
s5 = f"My name is {name} and I'm {age} years old"
print(s5)  # My name is Alice and I'm 30 years old

# Byte strings
b = b"Hello"
print(type(b))  # <class 'bytes'>

# String from bytes
text = b"Hello".decode('utf-8')
print(text)  # Hello

# Bytes from string
data = "Hello".encode('utf-8')
print(data)  # b'Hello'
```

#### 3.5.2 String Immutability

```python
text = "Hello"
# text[0] = 'h'  # TypeError: 'str' object does not support item assignment

# Must create new string
text = 'h' + text[1:]
print(text)  # hello

# String methods return new strings
original = "hello"
upper = original.upper()
print(original)  # hello (unchanged)
print(upper)     # HELLO (new string)
```

#### 3.5.3 String Methods

```python
text = "Hello, World!"

# Case conversion
print(text.upper())       # HELLO, WORLD!
print(text.lower())       # hello, world!
print(text.capitalize())  # Hello, world!
print(text.title())       # Hello, World!
print("hello world".swapcase())  # HELLO WORLD

# Stripping whitespace
s = "  hello  "
print(s.strip())   # "hello" (both ends)
print(s.lstrip())  # "hello  " (left)
print(s.rstrip())  # "  hello" (right)
print("***hello***".strip('*'))  # "hello" (custom chars)

# Splitting and joining
sentence = "Hello world from Python"
words = sentence.split()  # Split on whitespace
print(words)  # ['Hello', 'world', 'from', 'Python']

csv = "a,b,c,d"
items = csv.split(',')
print(items)  # ['a', 'b', 'c', 'd']

# Join
joined = ' '.join(['Hello', 'World'])
print(joined)  # Hello World

# Replace
text = "Hello World"
new_text = text.replace("World", "Python")
print(new_text)  # Hello Python
print(text.replace("o", "0"))  # Hell0 W0rld (all occurrences)
print(text.replace("o", "0", 1))  # Hell0 World (only first)

# Finding substrings
text = "Hello World"
print(text.find("World"))   # 6 (index of first occurrence)
print(text.find("Python"))  # -1 (not found)
print(text.rfind("o"))      # 7 (last occurrence)

# index() - like find() but raises ValueError if not found
print(text.index("World"))  # 6
# print(text.index("Python"))  # ValueError!

# Count occurrences
print("hello world".count("l"))  # 3

# startswith() and endswith()
filename = "document.pdf"
print(filename.startswith("doc"))   # True
print(filename.endswith(".pdf"))    # True
print(filename.endswith((".pdf", ".docx")))  # True (tuple of options)

# Padding
print("hello".center(11))      # "   hello   "
print("hello".ljust(10, '*'))  # "hello*****"
print("hello".rjust(10, '*'))  # "*****hello"
print("42".zfill(5))           # "00042" (zero-padding)
```

#### 3.5.4 Character Testing Methods

```python
# isdigit() - all characters are digits
print("123".isdigit())     # True
print("12.3".isdigit())    # False
print("abc".isdigit())     # False

# isalpha() - all characters are alphabetic
print("abc".isalpha())     # True
print("abc123".isalpha())  # False

# isalnum() - all characters are alphanumeric
print("abc123".isalnum())  # True
print("abc 123".isalnum()) # False (space)

# isspace() - all characters are whitespace
print("   ".isspace())     # True
print(" a ".isspace())     # False

# isupper() / islower()
print("HELLO".isupper())   # True
print("hello".islower())   # True
print("Hello".isupper())   # False

# istitle() - title case
print("Hello World".istitle())  # True
print("Hello world".istitle())  # False

# isidentifier() - valid Python identifier
print("variable_name".isidentifier())  # True
print("123abc".isidentifier())         # False
print("for".isidentifier())            # True (but it's a keyword!)

# Combined checks
text = "hello"
if text.isalpha() and text.islower():
    print("All lowercase letters")
```

#### 3.5.5 String Formatting

```python
name, age, salary = "Alice", 30, 75000.50

# Old % formatting (C-style)
print("Name: %s, Age: %d" % (name, age))
print("Salary: %.2f" % salary)  # 75000.50

# str.format() method
print("Name: {}, Age: {}".format(name, age))
print("Name: {0}, Age: {1}, Name again: {0}".format(name, age))
print("Name: {name}, Age: {age}".format(name=name, age=age))

# f-strings (Python 3.6+, preferred)
print(f"Name: {name}, Age: {age}")
print(f"Salary: {salary:.2f}")  # Format specifiers
print(f"Salary: ${salary:,.2f}")  # $75,000.50 (with comma separator)

# Expressions in f-strings
x, y = 10, 20
print(f"Sum: {x + y}")  # Sum: 30
print(f"Is x > y? {x > y}")  # Is x > y? False

# Calling functions in f-strings
print(f"Uppercase name: {name.upper()}")

# Format specifiers
value = 42
print(f"{value:05d}")   # 00042 (zero-padding)
print(f"{value:b}")     # 101010 (binary)
print(f"{value:x}")     # 2a (hexadecimal)
print(f"{value:o}")     # 52 (octal)

# Alignment
print(f"{'left':<10}|")   # "left      |"
print(f"{'right':>10}|")  # "     right|"
print(f"{'center':^10}|") # "  center  |"

# Percentage
ratio = 0.85
print(f"Success rate: {ratio:.1%}")  # Success rate: 85.0%

# Date formatting in f-strings
from datetime import datetime
now = datetime.now()
print(f"Current time: {now:%Y-%m-%d %H:%M:%S}")
```

#### 3.5.6 Slicing & Indexing

```python
text = "Hello, World!"

# Indexing
print(text[0])     # 'H'
print(text[-1])    # '!'
print(text[7])     # 'W'

# Slicing
print(text[0:5])   # 'Hello'
print(text[:5])    # 'Hello' (from start)
print(text[7:])    # 'World!' (to end)
print(text[::2])   # 'Hlo ol!' (every 2nd char)
print(text[::-1])  # '!dlroW ,olleH' (reverse)

# Substring checking
print("World" in text)      # True
print("Python" not in text) # True
```

#### 3.5.7 Escape Sequences

```python
# Common escape sequences
print("Line 1\nLine 2")         # Newline
print("Tab\there")              # Tab
print("Quote: \"Hello\"")       # Escape quote
print("Backslash: \\")          # Backslash
print("Carriage return: \r")    # Carriage return
print("Bell: \a")               # Bell/beep

# Unicode escape
print("\u0041")    # 'A' (Unicode code point)
print("\U0001F600") # 😀 (emoji)

# Raw strings ignore escapes
print(r"C:\new\test")  # C:\new\test (not C: ewline + est)
```

#### 3.5.8 String Encoding/Decoding

```python
# String to bytes (encoding)
text = "Hello, 世界"
utf8_bytes = text.encode('utf-8')
print(utf8_bytes)  # b'Hello, \xe4\xb8\x96\xe7\x95\x8c'

ascii_bytes = text.encode('ascii', errors='ignore')
print(ascii_bytes)  # b'Hello, ' (non-ASCII chars removed)

# Bytes to string (decoding)
decoded = utf8_bytes.decode('utf-8')
print(decoded)  # Hello, 世界

# Character code conversions
print(ord('A'))    # 65 (character to Unicode code point)
print(chr(65))     # 'A' (code point to character)
print(chr(0x1F600))  # 😀 (emoji from hex code)

# Check encoding
import sys
print(sys.getdefaultencoding())  # utf-8 (usually)
```

#### 3.5.9 String Gotchas

```python
# GOTCHA 1: Concatenation in loops (inefficient)
# BAD - creates new string each iteration
result = ""
for i in range(1000):
    result += str(i)  # Slow!

# GOOD - use join()
result = ''.join(str(i) for i in range(1000))

# GOTCHA 2: String interning
# Python interns small strings for optimization
a = "hello"
b = "hello"
print(a is b)  # True (same object!)

# But not always:
a = "hello world!"
b = "hello world!"
print(a is b)  # May be False (not interned)

# Always use == for string comparison, not is
print(a == b)  # True (correct way)

# GOTCHA 3: Immutability
text = "Hello"
text.upper()  # Returns new string, doesn't modify original
print(text)  # Still "Hello"
# Must reassign:
text = text.upper()
print(text)  # "HELLO"
```

---

## 4. CONTROL FLOW

### 4.1 if/elif/else Statements

```python
# Basic if statement
x = 10
if x > 5:
    print("x is greater than 5")

# if-else
age = 18
if age >= 18:
    print("Adult")
else:
    print("Minor")

# if-elif-else
score = 85
if score >= 90:
    grade = 'A'
elif score >= 80:
    grade = 'B'
elif score >= 70:
    grade = 'C'
elif score >= 60:
    grade = 'D'
else:
    grade = 'F'
print(f"Grade: {grade}")  # Grade: B

# Nested conditions
x, y = 10, 20
if x > 0:
    if y > 0:
        print("Both positive")
    else:
        print("x positive, y non-positive")
else:
    print("x non-positive")

# Multiple conditions
age, has_license = 18, True
if age >= 18 and has_license:
    print("Can drive")

# Truthiness in conditions
name = "Alice"
if name:  # Truthy (non-empty string)
    print(f"Hello, {name}")

items = []
if not items:  # Falsy (empty list)
    print("No items")
```

### 4.2 Ternary Operator

```python
# Syntax: value_if_true if condition else value_if_false
age = 20
status = "Adult" if age >= 18 else "Minor"
print(status)  # Adult

# In assignments
x = 10
y = 5
max_val = x if x > y else y
print(max_val)  # 10

# In function arguments
def greet(name, formal=False):
    greeting = "Good day" if formal else "Hi"
    print(f"{greeting}, {name}")

greet("Alice", True)   # Good day, Alice
greet("Bob", False)    # Hi, Bob

# Nested ternary (less readable, avoid if possible)
num = 0
result = "positive" if num > 0 else "negative" if num < 0 else "zero"
print(result)  # zero

# In list comprehensions
numbers = [1, 2, 3, 4, 5]
labels = ["even" if n % 2 == 0 else "odd" for n in numbers]
print(labels)  # ['odd', 'even', 'odd', 'even', 'odd']
```

### 4.3 match/case (Python 3.10+)

```python
# Basic pattern matching
def http_status(status):
    match status:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        case _:  # Wildcard (default)
            return "Unknown"

print(http_status(200))  # OK
print(http_status(403))  # Unknown

# Multiple patterns (OR)
def day_type(day):
    match day:
        case "Saturday" | "Sunday":
            return "Weekend"
        case "Monday" | "Tuesday" | "Wednesday" | "Thursday" | "Friday":
            return "Weekday"
        case _:
            return "Invalid day"

print(day_type("Saturday"))  # Weekend

# Pattern matching with guards
def categorize_number(n):
    match n:
        case 0:
            return "zero"
        case n if n < 0:
            return "negative"
        case n if n > 0:
            return "positive"

print(categorize_number(-5))  # negative

# Sequence patterns
def process_point(point):
    match point:
        case [0, 0]:
            return "Origin"
        case [0, y]:
            return f"Y-axis at {y}"
        case [x, 0]:
            return f"X-axis at {x}"
        case [x, y]:
            return f"Point at ({x}, {y})"
        case _:
            return "Invalid point"

print(process_point([0, 0]))   # Origin
print(process_point([0, 5]))   # Y-axis at 5
print(process_point([3, 4]))   # Point at (3, 4)

# Dictionary patterns
def process_user(user):
    match user:
        case {"name": name, "age": age} if age < 18:
            return f"{name} is a minor"
        case {"name": name, "age": age}:
            return f"{name} is {age} years old"
        case {"name": name}:
            return f"Name: {name}, age unknown"
        case _:
            return "Invalid user data"

print(process_user({"name": "Alice", "age": 15}))  # Alice is a minor
print(process_user({"name": "Bob", "age": 30}))    # Bob is 30 years old
```

### 4.4 for Loops

```python
# Basic for loop
fruits = ['apple', 'banana', 'cherry']
for fruit in fruits:
    print(fruit)

# range() - generate sequence of numbers
for i in range(5):      # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 8):   # 2, 3, 4, 5, 6, 7
    print(i)

for i in range(0, 10, 2):  # 0, 2, 4, 6, 8 (step=2)
    print(i)

for i in range(10, 0, -1):  # Countdown: 10, 9, 8, ..., 1
    print(i)

# enumerate() - get index and value
fruits = ['apple', 'banana', 'cherry']
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
# 0: apple
# 1: banana
# 2: cherry

# enumerate with custom start
for index, fruit in enumerate(fruits, start=1):
    print(f"{index}. {fruit}")
# 1. apple
# 2. banana
# 3. cherry

# zip() - iterate over multiple sequences in parallel
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]
jobs = ['Engineer', 'Designer', 'Manager']

for name, age, job in zip(names, ages, jobs):
    print(f"{name} is {age} years old and works as a {job}")

# zip stops at shortest sequence
list1 = [1, 2, 3]
list2 = ['a', 'b']
for x, y in zip(list1, list2):
    print(x, y)
# 1 a
# 2 b

# Iterating over dictionaries
person = {'name': 'Alice', 'age': 30, 'job': 'Engineer'}

# Keys (default)
for key in person:
    print(key)

# Explicit keys
for key in person.keys():
    print(key)

# Values
for value in person.values():
    print(value)

# Items (key-value pairs)
for key, value in person.items():
    print(f"{key}: {value}")

# Nested loops
for i in range(1, 4):
    for j in range(1, 4):
        print(f"{i} * {j} = {i*j}")

# Iterating over strings
for char in "Hello":
    print(char)
```

### 4.5 while Loops

```python
# Basic while loop
count = 0
while count < 5:
    print(count)
    count += 1

# User input loop
# while True:
#     response = input("Continue? (yes/no): ")
#     if response.lower() == 'no':
#         break

# Sentinel value
numbers = []
print("Enter numbers (0 to stop):")
# while (num := int(input())) != 0:  # Using walrus operator
#     numbers.append(num)

# Countdown
countdown = 5
while countdown > 0:
    print(countdown)
    countdown -= 1
print("Blast off!")

# While with complex condition
x, y = 0, 10
while x < 5 and y > 5:
    print(f"x={x}, y={y}")
    x += 1
    y -= 1
```

### 4.6 break Statement

```python
# Exit loop immediately
for i in range(10):
    if i == 5:
        break  # Stop loop when i is 5
    print(i)
# Prints: 0, 1, 2, 3, 4

# Search and stop
numbers = [1, 3, 5, 7, 9, 8, 6, 4]
target = 8
for num in numbers:
    if num == target:
        print(f"Found {target}!")
        break
else:
    print(f"{target} not found")  # Only runs if no break

# Break from nested loop (only exits inner loop)
for i in range(3):
    for j in range(3):
        if j == 1:
            break  # Only breaks inner loop
        print(f"i={i}, j={j}")

# To break outer loop, use flag or function return
found = False
for i in range(3):
    for j in range(3):
        if i == 1 and j == 1:
            found = True
            break
    if found:
        break
```

### 4.7 continue Statement

```python
# Skip to next iteration
for i in range(10):
    if i % 2 == 0:
        continue  # Skip even numbers
    print(i)
# Prints: 1, 3, 5, 7, 9

# Skip specific values
for num in range(1, 11):
    if num == 5:
        continue  # Skip 5
    print(num)

# Process only valid data
data = [1, -2, 3, -4, 5, -6]
for num in data:
    if num < 0:
        continue  # Skip negative numbers
    print(f"Processing: {num}")
```

### 4.8 pass Statement

```python
# Placeholder for future code
def not_implemented_yet():
    pass  # TODO: implement this

# Empty class
class MyClass:
    pass

# Empty if block
x = 10
if x > 5:
    pass  # Do nothing
else:
    print("x is small")

# Empty loop
for i in range(10):
    pass  # Loop does nothing

# Cannot have empty block without pass
# if True:  # SyntaxError!

if True:
    pass  # OK
```

### 4.9 else Clause with Loops

```python
# for...else: else block runs if loop completes without break
numbers = [1, 3, 5, 7, 9]
for num in numbers:
    if num % 2 == 0:
        print("Found even number!")
        break
else:
    print("No even numbers found")  # This runs (no break)

# while...else
count = 0
while count < 5:
    print(count)
    count += 1
else:
    print("Loop completed normally")  # Runs because no break

# Practical example: search
def find_value(lst, target):
    for item in lst:
        if item == target:
            print(f"Found {target}")
            break
    else:
        print(f"{target} not found")

find_value([1, 2, 3, 4], 3)  # Found 3
find_value([1, 2, 3, 4], 5)  # 5 not found

# else doesn't run if break is executed
for i in range(5):
    if i == 3:
        break
    print(i)
else:
    print("This won't print")  # Skipped due to break
```

### 4.10 Loop Control Patterns

```python
# Early exit pattern
def process_items(items):
    for item in items:
        if not is_valid(item):
            print(f"Invalid item: {item}")
            return False
        process(item)
    return True

# Sentinel value pattern
# while (line := input("Enter line: ")) != "quit":
#     print(f"You entered: {line}")

# Loop and a half pattern
# while True:
#     data = get_data()
#     if not data:
#         break
#     process(data)

# Counting pattern
count = 0
for item in collection:
    if condition(item):
        count += 1
# Better: count = sum(1 for item in collection if condition(item))

# Finding pattern
result = None
for item in collection:
    if matches(item):
        result = item
        break
# Better: result = next((item for item in collection if matches(item)), None)
```

---

## 5. FUNCTIONS

### 5.1 Definition & Calling

```python
# Basic function
def greet():
    print("Hello, World!")

greet()  # Hello, World!

# Function with parameters
def greet_person(name):
    print(f"Hello, {name}!")

greet_person("Alice")  # Hello, Alice!

# Function with return value
def add(a, b):
    return a + b

result = add(3, 5)
print(result)  # 8

# Multiple return values (returns tuple)
def divide_with_remainder(a, b):
    quotient = a // b
    remainder = a % b
    return quotient, remainder

q, r = divide_with_remainder(17, 5)
print(f"Quotient: {q}, Remainder: {r}")  # Quotient: 3, Remainder: 2

# Early return
def is_adult(age):
    if age >= 18:
        return True
    return False

# No return statement → returns None
def no_return():
    x = 5

result = no_return()
print(result)  # None
```

### 5.2 Docstrings

```python
def calculate_area(length, width):
    """
    Calculate the area of a rectangle.
    
    Args:
        length (float): The length of the rectangle
        width (float): The width of the rectangle
    
    Returns:
        float: The area of the rectangle
    
    Example:
        >>> calculate_area(5, 3)
        15
    """
    return length * width

# Access docstring
print(calculate_area.__doc__)
help(calculate_area)

# One-line docstring
def square(n):
    """Return the square of n."""
    return n ** 2
```

### 5.3 Parameters & Arguments

#### 5.3.1 Positional & Keyword Arguments

```python
def describe_person(name, age, job):
    print(f"{name} is {age} years old and works as a {job}")

# Positional arguments (order matters)
describe_person("Alice", 30, "Engineer")

# Keyword arguments (order doesn't matter)
describe_person(job="Engineer", name="Alice", age=30)

# Mixed (positional first, then keyword)
describe_person("Alice", job="Engineer", age=30)

# Can't have positional after keyword
# describe_person(name="Alice", 30, "Engineer")  # SyntaxError!
```

#### 5.3.2 Default Parameters

```python
def greet(name, greeting="Hello"):
    print(f"{greeting}, {name}!")

greet("Alice")              # Hello, Alice!
greet("Bob", "Hi")          # Hi, Bob!
greet("Charlie", greeting="Hey")  # Hey, Charlie!

# Multiple default parameters
def create_profile(name, age=None, job=None):
    profile = {"name": name}
    if age is not None:
        profile["age"] = age
    if job is not None:
        profile["job"] = job
    return profile

print(create_profile("Alice"))                    # {'name': 'Alice'}
print(create_profile("Bob", age=30))              # {'name': 'Bob', 'age': 30}
print(create_profile("Charlie", job="Engineer"))  # {'name': 'Charlie', 'job': 'Engineer'}

# IMPORTANT: Default values evaluated once at definition time
def append_to(element, target=[]):  # DANGEROUS!
    target.append(element)
    return target

print(append_to(1))  # [1]
print(append_to(2))  # [1, 2] - same list!
print(append_to(3))  # [1, 2, 3] - still same list!

# FIX: Use None as default
def append_to_safe(element, target=None):
    if target is None:
        target = []
    target.append(element)
    return target

print(append_to_safe(1))  # [1]
print(append_to_safe(2))  # [2] - new list each time
```

#### 5.3.3 *args (Variable Positional Arguments)

```python
# Accept any number of positional arguments
def sum_all(*args):
    return sum(args)

print(sum_all(1, 2, 3))           # 6
print(sum_all(1, 2, 3, 4, 5))     # 15
print(sum_all())                  # 0

# args is a tuple
def print_args(*args):
    print(type(args))  # <class 'tuple'>
    for i, arg in enumerate(args):
        print(f"Argument {i}: {arg}")

print_args("a", "b", "c")

# Combining with regular parameters
def greet_all(greeting, *names):
    for name in names:
        print(f"{greeting}, {name}!")

greet_all("Hello", "Alice", "Bob", "Charlie")
# Hello, Alice!
# Hello, Bob!
# Hello, Charlie!

# Unpacking with *
def add_three(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
result = add_three(*numbers)  # Unpacks list
print(result)  # 6
```

#### 5.3.4 **kwargs (Variable Keyword Arguments)

```python
# Accept any number of keyword arguments
def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=30, job="Engineer")
# name: Alice
# age: 30
# job: Engineer

# kwargs is a dictionary
def show_kwargs(**kwargs):
    print(type(kwargs))  # <class 'dict'>
    print(kwargs)

show_kwargs(a=1, b=2, c=3)  # {'a': 1, 'b': 2, 'c': 3}

# Combining with regular and *args
def complex_function(required, *args, **kwargs):
    print(f"Required: {required}")
    print(f"Args: {args}")
    print(f"Kwargs: {kwargs}")

complex_function(1, 2, 3, name="Alice", age=30)
# Required: 1
# Args: (2, 3)
# Kwargs: {'name': 'Alice', 'age': 30}

# Unpacking with **
def create_user(name, age, job):
    return f"{name}, {age}, {job}"

user_data = {"name": "Alice", "age": 30, "job": "Engineer"}
result = create_user(**user_data)  # Unpacks dict
print(result)  # Alice, 30, Engineer
```

#### 5.3.5 Positional-Only Parameters (Python 3.8+)

```python
# Parameters before / are positional-only
def greet(name, /, greeting="Hello"):
    print(f"{greeting}, {name}!")

greet("Alice")                    # OK
greet("Alice", "Hi")              # OK
greet("Alice", greeting="Hey")    # OK
# greet(name="Alice")             # TypeError! name is positional-only

# Useful for clarity and allowing parameter name changes
def calculate(a, b, /):
    return a + b

calculate(3, 5)      # OK
# calculate(a=3, b=5)  # TypeError!

# Combining positional-only with other parameters
def func(pos_only, /, pos_or_kwd, *, kwd_only):
    print(pos_only, pos_or_kwd, kwd_only)

func(1, 2, kwd_only=3)        # OK
func(1, pos_or_kwd=2, kwd_only=3)  # OK
# func(pos_only=1, pos_or_kwd=2, kwd_only=3)  # TypeError!
```

#### 5.3.6 Keyword-Only Parameters

```python
# Parameters after * are keyword-only
def create_user(name, age, *, job, salary):
    return f"{name}, {age}, {job}, {salary}"

# Must use keywords for job and salary
result = create_user("Alice", 30, job="Engineer", salary=80000)
print(result)

# This fails:
# create_user("Alice", 30, "Engineer", 80000)  # TypeError!

# * alone means all following parameters are keyword-only
def func(a, b, *, c, d):
    print(a, b, c, d)

func(1, 2, c=3, d=4)  # OK
# func(1, 2, 3, 4)      # TypeError!

# Combining with defaults
def configure(*, debug=False, verbose=False):
    print(f"Debug: {debug}, Verbose: {verbose}")

configure(debug=True)  # Debug: True, Verbose: False
configure()            # Debug: False, Verbose: False
# configure(True)        # TypeError!
```

#### 5.3.7 Parameter Ordering Rules

```python
# Complete ordering (all types):
# 1. Positional-only (before /)
# 2. Regular positional or keyword
# 3. *args
# 4. Keyword-only (after * or *args)
# 5. **kwargs

def full_example(pos_only, /, pos_or_kwd, *args, kwd_only, **kwargs):
    print(f"pos_only: {pos_only}")
    print(f"pos_or_kwd: {pos_or_kwd}")
    print(f"args: {args}")
    print(f"kwd_only: {kwd_only}")
    print(f"kwargs: {kwargs}")

full_example(1, 2, 3, 4, kwd_only=5, extra1=6, extra2=7)
# pos_only: 1
# pos_or_kwd: 2
# args: (3, 4)
# kwd_only: 5
# kwargs: {'extra1': 6, 'extra2': 7}
```

### 5.4 Scope (LEGB Rule)

```python
# LEGB: Local, Enclosing, Global, Built-in

# Global scope
x = "global"

def outer():
    # Enclosing scope
    x = "enclosing"
    
    def inner():
        # Local scope
        x = "local"
        print(f"Inner: {x}")  # local
    
    inner()
    print(f"Outer: {x}")  # enclosing

outer()
print(f"Global: {x}")  # global

# Built-in scope (last resort)
print(len([1, 2, 3]))  # len is built-in

# global keyword - modify global variable
count = 0

def increment():
    global count  # Without this, would create local variable
    count += 1

increment()
increment()
print(count)  # 2

# nonlocal keyword - modify enclosing scope variable
def outer():
    x = 0
    
    def inner():
        nonlocal x  # Modify enclosing scope's x
        x += 1
    
    inner()
    inner()
    print(f"x = {x}")  # 2

outer()

# Closures
def make_multiplier(n):
    def multiplier(x):
        return x * n  # n from enclosing scope
    return multiplier

times_3 = make_multiplier(3)
times_5 = make_multiplier(5)

print(times_3(10))  # 30
print(times_5(10))  # 50

# Free variables
def outer(x):
    def inner(y):
        return x + y  # x is free variable in inner
    return inner

add_5 = outer(5)
print(add_5(3))  # 8
print(add_5.__code__.co_freevars)  # ('x',)
```

### 5.5 Lambda Functions

```python
# Basic lambda (anonymous function)
square = lambda x: x ** 2
print(square(5))  # 25

# Lambda with multiple parameters
add = lambda a, b: a + b
print(add(3, 5))  # 8

# Lambda in sorted() with key parameter
pairs = [(1, 'one'), (3, 'three'), (2, 'two')]
sorted_pairs = sorted(pairs, key=lambda pair: pair[1])
print(sorted_pairs)  # [(1, 'one'), (3, 'three'), (2, 'two')]

# Lambda with map()
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# Lambda with filter()
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4]

# Limitations of lambda:
# - Single expression only (no statements)
# - No type annotations
# - Less readable for complex logic

# DON'T DO THIS (use def instead):
# complex_lambda = lambda x: (x**2 if x > 0 else -x**2)

# DO THIS:
def complex_function(x):
    if x > 0:
        return x ** 2
    else:
        return -x ** 2

# Good lambda uses: short, simple operations
names = ['Alice', 'bob', 'Charlie']
sorted_names = sorted(names, key=lambda s: s.lower())
print(sorted_names)  # ['Alice', 'bob', 'Charlie']
```

### 5.6 First-Class Functions & Higher-Order Functions

```python
# Functions are first-class objects (can be assigned, passed, returned)

# Assign function to variable
def greet(name):
    return f"Hello, {name}!"

say_hello = greet
print(say_hello("Alice"))  # Hello, Alice!

# Pass function as argument
def apply_operation(func, value):
    return func(value)

def square(x):
    return x ** 2

result = apply_operation(square, 5)
print(result)  # 25

# Return function from function
def make_multiplier(n):
    def multiplier(x):
        return x * n
    return multiplier

times_2 = make_multiplier(2)
times_3 = make_multiplier(3)
print(times_2(5))  # 10
print(times_3(5))  # 15

# Store functions in data structures
operations = {
    'add': lambda a, b: a + b,
    'subtract': lambda a, b: a - b,
    'multiply': lambda a, b: a * b,
    'divide': lambda a, b: a / b if b != 0 else None
}

print(operations['add'](10, 5))      # 15
print(operations['multiply'](10, 5)) # 50

# Higher-order functions
def apply_twice(func, value):
    return func(func(value))

def add_one(x):
    return x + 1

result = apply_twice(add_one, 5)  # add_one(add_one(5))
print(result)  # 7
```

### 5.7 Decorators

#### 5.7.1 Basic Decorators

```python
# Decorator is a function that wraps another function
def uppercase_decorator(func):
    def wrapper():
        result = func()
        return result.upper()
    return wrapper

@uppercase_decorator
def greet():
    return "hello, world"

print(greet())  # HELLO, WORLD

# Equivalent to:
# greet = uppercase_decorator(greet)

# Decorator with arguments
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")

say_hello()
# Hello!
# Hello!
# Hello!

# Decorator that accepts function arguments
def trace(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}({args}, {kwargs})")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@trace
def add(a, b):
    return a + b

add(3, 5)
# Calling add((3, 5), {})
# add returned 8
```

#### 5.7.2 functools.wraps

```python
from functools import wraps

# Without @wraps
def decorator_without_wraps(func):
    def wrapper(*args, **kwargs):
        """Wrapper docstring"""
        return func(*args, **kwargs)
    return wrapper

@decorator_without_wraps
def greet():
    """Greet function docstring"""
    return "Hello"

print(greet.__name__)  # wrapper (lost original name!)
print(greet.__doc__)   # Wrapper docstring (lost original doc!)

# With @wraps (preserves metadata)
def decorator_with_wraps(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        """Wrapper docstring"""
        return func(*args, **kwargs)
    return wrapper

@decorator_with_wraps
def hello():
    """Hello function docstring"""
    return "Hello"

print(hello.__name__)  # hello (preserved!)
print(hello.__doc__)   # Hello function docstring (preserved!)
```

#### 5.7.3 Practical Decorator Examples

```python
import time
from functools import wraps

# Timing decorator
def timing_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timing_decorator
def slow_function():
    time.sleep(1)
    return "Done"

slow_function()  # slow_function took 1.0001 seconds

# Logging decorator
def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} completed")
        return result
    return wrapper

@log_calls
def process_data(data):
    return data.upper()

process_data("hello")

# Caching decorator (memoization)
def memoize(func):
    cache = {}
    @wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(10))  # Much faster with caching!

# Stacking decorators (applied bottom-up)
@trace
@timing_decorator
def calculate(x, y):
    time.sleep(0.5)
    return x + y

calculate(3, 5)
```

#### 5.7.4 Class Decorators

```python
# Decorating classes
def singleton(cls):
    """Ensure only one instance of class exists"""
    instances = {}
    @wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class Database:
    def __init__(self):
        print("Creating database connection")

db1 = Database()  # Creating database connection
db2 = Database()  # No output (returns existing instance)
print(db1 is db2)  # True
```

### 5.8 functools.partial

```python
from functools import partial

# Create new function with some arguments pre-filled
def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))  # 25
print(cube(5))    # 125

# Practical use with sorted()
def compare_by_age(person):
    return person['age']

people = [
    {'name': 'Alice', 'age': 30},
    {'name': 'Bob', 'age': 25},
    {'name': 'Charlie', 'age': 35}
]

sorted_people = sorted(people, key=lambda p: p['age'])
# Or with partial:
# from operator import itemgetter
# sorted_people = sorted(people, key=itemgetter('age'))

# Partial with multiple arguments
def greet(greeting, name, punctuation):
    return f"{greeting}, {name}{punctuation}"

casual_greet = partial(greet, "Hey", punctuation="!")
print(casual_greet("Alice"))  # Hey, Alice!

formal_greet = partial(greet, "Good day", punctuation=".")
print(formal_greet("Bob"))  # Good day, Bob.
```

### 5.9 Generator Functions

```python
# Generator function (uses yield)
def count_up_to(n):
    i = 1
    while i <= n:
        yield i
        i += 1

# Generators are lazy (values produced on demand)
counter = count_up_to(5)
print(type(counter))  # <class 'generator'>

print(next(counter))  # 1
print(next(counter))  # 2

# Iterate over generator
for num in count_up_to(5):
    print(num)  # 1, 2, 3, 4, 5

# Generator exhausts after one use
gen = count_up_to(3)
print(list(gen))  # [1, 2, 3]
print(list(gen))  # [] (exhausted!)

# yield from (Python 3.3+)
def chain(*iterables):
    for iterable in iterables:
        yield from iterable

result = list(chain([1, 2], [3, 4], [5, 6]))
print(result)  # [1, 2, 3, 4, 5, 6]

# Infinite generator
def infinite_counter():
    n = 0
    while True:
        yield n
        n += 1

# Take first 10 values
from itertools import islice
first_10 = list(islice(infinite_counter(), 10))
print(first_10)  # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Generator methods: send(), throw(), close()
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value

acc = accumulator()
next(acc)  # Initialize generator
print(acc.send(10))  # 10
print(acc.send(20))  # 30
print(acc.send(5))   # 35
```

### 5.10 Recursive Functions

```python
# Basic recursion
def factorial(n):
    if n <= 1:  # Base case
        return 1
    return n * factorial(n - 1)  # Recursive case

print(factorial(5))  # 120

# Fibonacci
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(10))  # 55

# Recursion with accumulator (tail recursion style)
def factorial_acc(n, acc=1):
    if n <= 1:
        return acc
    return factorial_acc(n - 1, n * acc)

print(factorial_acc(5))  # 120

# Recursion with lists
def sum_list(lst):
    if not lst:
        return 0
    return lst[0] + sum_list(lst[1:])

print(sum_list([1, 2, 3, 4, 5]))  # 15

# Binary search (recursive)
def binary_search(lst, target, left=0, right=None):
    if right is None:
        right = len(lst) - 1
    
    if left > right:
        return -1  # Not found
    
    mid = (left + right) // 2
    if lst[mid] == target:
        return mid
    elif lst[mid] < target:
        return binary_search(lst, target, mid + 1, right)
    else:
        return binary_search(lst, target, left, mid - 1)

numbers = [1, 3, 5, 7, 9, 11, 13]
print(binary_search(numbers, 7))  # 3
print(binary_search(numbers, 6))  # -1

# Maximum recursion depth
import sys
print(sys.getrecursionlimit())  # Usually 1000
# sys.setrecursionlimit(2000)  # Increase if needed (use carefully!)
```

---

## 6. OBJECT-ORIENTED PROGRAMMING

### 6.1 Classes Fundamentals

```python
# Basic class definition
class Person:
    pass

# Create instance
p = Person()
print(type(p))  # <class '__main__.Person'>

# Class with __init__ constructor
class Person:
    def __init__(self, name, age):
        self.name = name  # Instance attribute
        self.age = age
    
    def greet(self):  # Instance method
        return f"Hello, I'm {self.name}"

# Create instances
alice = Person("Alice", 30)
bob = Person("Bob", 25)

print(alice.name)     # Alice
print(alice.greet())  # Hello, I'm Alice
print(bob.greet())    # Hello, I'm Bob

# Class attributes (shared by all instances)
class Dog:
    species = "Canis familiaris"  # Class attribute
    
    def __init__(self, name, breed):
        self.name = name    # Instance attribute
        self.breed = breed

dog1 = Dog("Buddy", "Labrador")
dog2 = Dog("Max", "Poodle")

print(dog1.species)  # Canis familiaris
print(dog2.species)  # Canis familiaris
print(Dog.species)   # Canis familiaris

# GOTCHA: Mutable class attributes
class MyClass:
    shared_list = []  # DANGEROUS!
    
    def add_item(self, item):
        self.shared_list.append(item)

obj1 = MyClass()
obj2 = MyClass()

obj1.add_item(1)
obj2.add_item(2)

print(obj1.shared_list)  # [1, 2] - shared!
print(obj2.shared_list)  # [1, 2] - same list!

# FIX: Use instance attribute
class MyClassFixed:
    def __init__(self):
        self.instance_list = []  # Each instance gets own list
    
    def add_item(self, item):
        self.instance_list.append(item)
```

### 6.2 Methods

#### 6.2.1 Instance Methods

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):  # Instance method (has self parameter)
        return 3.14159 * self.radius ** 2
    
    def circumference(self):
        return 2 * 3.14159 * self.radius

c = Circle(5)
print(c.area())           # 78.53975
print(c.circumference())  # 31.4159
```

#### 6.2.2 Class Methods

```python
class Employee:
    company = "TechCorp"  # Class attribute
    
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
    
    @classmethod
    def from_string(cls, emp_str):
        """Alternative constructor using class method"""
        name, salary = emp_str.split('-')
        return cls(name, int(salary))
    
    @classmethod
    def change_company(cls, new_company):
        """Modify class attribute"""
        cls.company = new_company

# Regular construction
emp1 = Employee("Alice", 50000)

# Using class method as alternative constructor
emp2 = Employee.from_string("Bob-60000")
print(emp2.name, emp2.salary)  # Bob 60000

# Modify class attribute
Employee.change_company("NewCorp")
print(emp1.company)  # NewCorp
print(emp2.company)  # NewCorp
```

#### 6.2.3 Static Methods

```python
class MathUtils:
    @staticmethod
    def add(a, b):
        """No access to instance or class"""
        return a + b
    
    @staticmethod
    def is_even(n):
        return n % 2 == 0

# Call without creating instance
print(MathUtils.add(5, 3))    # 8
print(MathUtils.is_even(10))  # True

# Can also call from instance
utils = MathUtils()
print(utils.add(2, 3))  # 5
```

#### 6.2.4 Properties

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius
    
    @property
    def celsius(self):
        """Getter"""
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        """Setter with validation"""
        if value < -273.15:
            raise ValueError("Temperature below absolute zero!")
        self._celsius = value
    
    @property
    def fahrenheit(self):
        """Computed property"""
        return self._celsius * 9/5 + 32
    
    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9
    
    @celsius.deleter
    def celsius(self):
        """Deleter"""
        print("Deleting celsius")
        del self._celsius

temp = Temperature(25)
print(temp.celsius)     # 25 (calls getter)
print(temp.fahrenheit)  # 77.0

temp.celsius = 30      # Calls setter
print(temp.fahrenheit)  # 86.0

temp.fahrenheit = 32   # Calls fahrenheit setter
print(temp.celsius)    # 0.0

# del temp.celsius      # Calls deleter

# Properties allow attribute-style access with method logic
```

### 6.3 Inheritance

#### 6.3.1 Single Inheritance

```python
# Base class (parent/superclass)
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return "Some sound"
    
    def info(self):
        return f"I'm {self.name}"

# Derived class (child/subclass)
class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)  # Call parent constructor
        self.breed = breed
    
    def speak(self):  # Override parent method
        return "Woof!"
    
    def fetch(self):  # New method specific to Dog
        return f"{self.name} is fetching"

dog = Dog("Buddy", "Labrador")
print(dog.name)      # Buddy (inherited attribute)
print(dog.speak())   # Woof! (overridden method)
print(dog.info())    # I'm Buddy (inherited method)
print(dog.fetch())   # Buddy is fetching (new method)
print(isinstance(dog, Dog))     # True
print(isinstance(dog, Animal))  # True (dog is also an Animal)
```

#### 6.3.2 super()

```python
class Vehicle:
    def __init__(self, brand, model):
        self.brand = brand
        self.model = model
    
    def info(self):
        return f"{self.brand} {self.model}"

class Car(Vehicle):
    def __init__(self, brand, model, num_doors):
        super().__init__(brand, model)  # Call parent __init__
        self.num_doors = num_doors
    
    def info(self):
        parent_info = super().info()  # Call parent method
        return f"{parent_info} with {self.num_doors} doors"

car = Car("Toyota", "Camry", 4)
print(car.info())  # Toyota Camry with 4 doors
```

#### 6.3.3 Multiple Inheritance

```python
class Flyer:
    def fly(self):
        return "Flying high!"

class Swimmer:
    def swim(self):
        return "Swimming fast!"

# Multiple inheritance
class Duck(Flyer, Swimmer):
    def quack(self):
        return "Quack!"

duck = Duck()
print(duck.fly())    # Flying high!
print(duck.swim())   # Swimming fast!
print(duck.quack())  # Quack!
```

#### 6.3.4 Method Resolution Order (MRO)

```python
class A:
    def process(self):
        return "A"

class B(A):
    def process(self):
        return "B"

class C(A):
    def process(self):
        return "C"

class D(B, C):  # Multiple inheritance
    pass

d = D()
print(d.process())  # B (from B, not C)

# Check MRO
print(D.__mro__)
# (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, 
#  <class '__main__.A'>, <class 'object'>)

print(D.mro())
# [<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, 
#  <class '__main__.A'>, <class 'object'>]

# C3 Linearization: D -> B -> C -> A -> object
# Method lookup follows this order left-to-right
```

#### 6.3.5 Cooperative Multiple Inheritance

```python
class Base:
    def __init__(self):
        print("Base.__init__")

class A(Base):
    def __init__(self):
        print("A.__init__")
        super().__init__()

class B(Base):
    def __init__(self):
        print("B.__init__")
        super().__init__()

class C(A, B):
    def __init__(self):
        print("C.__init__")
        super().__init__()

c = C()
# C.__init__
# A.__init__
# B.__init__
# Base.__init__

# super() follows MRO, ensuring each class called once
print(C.__mro__)
# (<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>, 
#  <class '__main__.Base'>, <class 'object'>)
```

#### 6.3.6 Abstract Base Classes

```python
from abc import ABC, abstractmethod

# Abstract base class
class Shape(ABC):
    @abstractmethod
    def area(self):
        """Subclasses must implement this"""
        pass
    
    @abstractmethod
    def perimeter(self):
        pass
    
    def description(self):
        """Concrete method (has implementation)"""
        return "I'm a shape"

# Can't instantiate abstract class
# shape = Shape()  # TypeError!

# Concrete subclass
class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)

rect = Rectangle(5, 3)
print(rect.area())         # 15
print(rect.perimeter())    # 16
print(rect.description())  # I'm a shape

# Must implement all abstract methods
# class BadCircle(Shape):
#     def area(self):
#         return 0
#     # Missing perimeter()!
# 
# c = BadCircle()  # TypeError!
```

### 6.4 Encapsulation

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner           # Public
        self._account_number = "123"  # Protected (convention: single underscore)
        self.__balance = balance      # Private (name mangling: double underscore)
    
    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount
    
    def withdraw(self, amount):
        if 0 < amount <= self.__balance:
            self.__balance -= amount
            return True
        return False
    
    def get_balance(self):
        return self.__balance

account = BankAccount("Alice", 1000)

# Public attribute (accessible)
print(account.owner)  # Alice

# Protected attribute (accessible but discouraged)
print(account._account_number)  # 123

# Private attribute (name-mangled, not directly accessible)
# print(account.__balance)  # AttributeError!

# Access through public method
print(account.get_balance())  # 1000

# Name mangling: Python renames __balance to _BankAccount__balance
print(account._BankAccount__balance)  # 1000 (possible but discouraged!)

# Property-based access control (better approach)
class SecureAccount:
    def __init__(self, balance):
        self._balance = balance
    
    @property
    def balance(self):
        """Read-only property"""
        return self._balance
    
    # No setter, so balance can't be modified directly

secure = SecureAccount(1000)
print(secure.balance)  # 1000
# secure.balance = 500  # AttributeError: can't set attribute
```

### 6.5 Magic/Dunder Methods

#### 6.5.1 Object Creation & Initialization

```python
class Person:
    def __new__(cls, *args, **kwargs):
        """Create instance (rarely overridden)"""
        print("__new__ called")
        instance = super().__new__(cls)
        return instance
    
    def __init__(self, name, age):
        """Initialize instance"""
        print("__init__ called")
        self.name = name
        self.age = age
    
    def __del__(self):
        """Destructor (called when object is garbage collected)"""
        print(f"Deleting {self.name}")

p = Person("Alice", 30)
# __new__ called
# __init__ called

# del p  # Deleting Alice (may not be immediate)
```

#### 6.5.2 String Representations

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __repr__(self):
        """Unambiguous representation (for developers)"""
        return f"Person('{self.name}', {self.age})"
    
    def __str__(self):
        """Human-readable representation"""
        return f"{self.name}, age {self.age}"
    
    def __format__(self, format_spec):
        """Custom formatting"""
        if format_spec == 'short':
            return self.name
        elif format_spec == 'long':
            return f"{self.name} ({self.age} years old)"
        return str(self)

p = Person("Alice", 30)
print(repr(p))  # Person('Alice', 30)
print(str(p))   # Alice, age 30
print(f"{p}")   # Alice, age 30
print(f"{p:short}")  # Alice
print(f"{p:long}")   # Alice (30 years old)
```

#### 6.5.3 Comparison Operators

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __eq__(self, other):
        """Equal to: =="""
        if not isinstance(other, Person):
            return NotImplemented
        return self.age == other.age
    
    def __ne__(self, other):
        """Not equal to: !="""
        return not self.__eq__(other)
    
    def __lt__(self, other):
        """Less than: <"""
        if not isinstance(other, Person):
            return NotImplemented
        return self.age < other.age
    
    def __le__(self, other):
        """Less than or equal: <="""
        return self.__lt__(other) or self.__eq__(other)
    
    def __gt__(self, other):
        """Greater than: >"""
        if not isinstance(other, Person):
            return NotImplemented
        return self.age > other.age
    
    def __ge__(self, other):
        """Greater than or equal: >="""
        return self.__gt__(other) or self.__eq__(other)

alice = Person("Alice", 30)
bob = Person("Bob", 25)

print(alice == bob)  # False
print(alice > bob)   # True
print(bob < alice)   # True

# Sort list of persons
people = [alice, bob, Person("Charlie", 35)]
sorted_people = sorted(people)  # Sorts by age
for p in sorted_people:
    print(p.name, p.age)

# Alternative: use functools.total_ordering
from functools import total_ordering

@total_ordering
class SimplePerson:
    def __init__(self, age):
        self.age = age
    
    def __eq__(self, other):
        return self.age == other.age
    
    def __lt__(self, other):
        return self.age < other.age
    # total_ordering generates other comparison methods!
```

#### 6.5.4 Arithmetic Operators

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        """Addition: +"""
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other):
        """Subtraction: -"""
        return Vector(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar):
        """Multiplication: *"""
        return Vector(self.x * scalar, self.y * scalar)
    
    def __truediv__(self, scalar):
        """Division: /"""
        return Vector(self.x / scalar, self.y / scalar)
    
    def __floordiv__(self, scalar):
        """Floor division: //"""
        return Vector(self.x // scalar, self.y // scalar)
    
    def __mod__(self, scalar):
        """Modulo: %"""
        return Vector(self.x % scalar, self.y % scalar)
    
    def __pow__(self, power):
        """Exponentiation: **"""
        return Vector(self.x ** power, self.y ** power)
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(3, 4)
v2 = Vector(1, 2)

print(v1 + v2)   # Vector(4, 6)
print(v1 - v2)   # Vector(2, 2)
print(v1 * 2)    # Vector(6, 8)
print(v1 / 2)    # Vector(1.5, 2.0)
print(v1 ** 2)   # Vector(9, 16)
```

#### 6.5.5 Container Emulation

```python
class CustomList:
    def __init__(self, data=None):
        self._data = data if data is not None else []
    
    def __len__(self):
        """len(obj)"""
        return len(self._data)
    
    def __getitem__(self, index):
        """obj[index]"""
        return self._data[index]
    
    def __setitem__(self, index, value):
        """obj[index] = value"""
        self._data[index] = value
    
    def __delitem__(self, index):
        """del obj[index]"""
        del self._data[index]
    
    def __contains__(self, item):
        """item in obj"""
        return item in self._data
    
    def __iter__(self):
        """for item in obj"""
        return iter(self._data)
    
    def __repr__(self):
        return f"CustomList({self._data})"

cl = CustomList([1, 2, 3, 4, 5])

print(len(cl))      # 5
print(cl[2])        # 3
cl[2] = 99
print(cl[2])        # 99
del cl[0]
print(cl)           # CustomList([2, 99, 4, 5])
print(99 in cl)     # True

for item in cl:
    print(item)  # 2, 99, 4, 5
```

#### 6.5.6 Callable Objects

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor
    
    def __call__(self, x):
        """Makes instance callable like a function"""
        return x * self.factor

times_5 = Multiplier(5)
print(times_5(3))   # 15
print(times_5(10))  # 50

# Check if callable
print(callable(times_5))  # True
```

#### 6.5.7 Context Managers

```python
class FileHandler:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        """Called when entering 'with' block"""
        print(f"Opening {self.filename}")
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_value, traceback):
        """Called when exiting 'with' block"""
        print(f"Closing {self.filename}")
        if self.file:
            self.file.close()
        # Return False to propagate exceptions, True to suppress
        return False

# Use with 'with' statement
with FileHandler('test.txt', 'w') as f:
    f.write("Hello, World!")
# File automatically closed after 'with' block
```

#### 6.5.8 Attribute Access

```python
class DynamicAttributes:
    def __init__(self):
        self._data = {}
    
    def __getattr__(self, name):
        """Called when attribute not found normally"""
        print(f"Getting {name}")
        return self._data.get(name, f"No attribute '{name}'")
    
    def __setattr__(self, name, value):
        """Called on all attribute assignments"""
        if name == '_data':
            super().__setattr__(name, value)
        else:
            print(f"Setting {name} = {value}")
            self._data[name] = value
    
    def __delattr__(self, name):
        """Called when deleting attribute"""
        print(f"Deleting {name}")
        if name in self._data:
            del self._data[name]
    
    def __getattribute__(self, name):
        """Called for ALL attribute access"""
        print(f"Accessing {name}")
        return super().__getattribute__(name)

obj = DynamicAttributes()
obj.x = 10  # Setting x = 10
print(obj.x)  # Accessing _data, Getting x -> 10
# del obj.x  # Accessing _data, Deleting x
```

#### 6.5.9 Hashing

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    
    def __hash__(self):
        """Make object hashable (can be dict key or in set)"""
        return hash((self.x, self.y))
    
    def __repr__(self):
        return f"Point({self.x}, {self.y})"

p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = Point(3, 4)

# Can use as dict keys
points_dict = {p1: "first", p3: "second"}
print(points_dict[p2])  # "first" (p2 == p1)

# Can add to sets
points_set = {p1, p2, p3}
print(points_set)  # {Point(1, 2), Point(3, 4)} (p1 and p2 are equal)
```

#### 6.5.10 Boolean Context

```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance
    
    def __bool__(self):
        """Determines truthiness of object"""
        return self.balance > 0

account1 = BankAccount(100)
account2 = BankAccount(0)

if account1:
    print("Account 1 has funds")  # Prints

if not account2:
    print("Account 2 is empty")  # Prints

# Without __bool__, __len__ is used; if neither, always truthy
```

### 6.6 Advanced OOP

#### 6.6.1 __slots__

```python
# Normal class: each instance has __dict__ (overhead)
class PersonNormal:
    def __init__(self, name, age):
        self.name = name
        self.age = age

# With __slots__: fixed attributes, memory optimization
class PersonSlots:
    __slots__ = ['name', 'age']
    
    def __init__(self, name, age):
        self.name = name
        self.age = age

p1 = PersonNormal("Alice", 30)
p2 = PersonSlots("Bob", 25)

# Normal class allows dynamic attributes
p1.new_attr = "works"

# Slots class doesn't
# p2.new_attr = "fails"  # AttributeError!

# Check memory (slots is more efficient)
import sys
print(sys.getsizeof(p1.__dict__))  # Larger
# print(sys.getsizeof(p2.__dict__))  # No __dict__!

# Use __slots__ when:
# - Creating many instances
# - Fixed set of attributes
# - Memory optimization needed
```

#### 6.6.2 Dataclasses (Python 3.7+)

```python
from dataclasses import dataclass, field

# Automatically generates __init__, __repr__, __eq__, etc.
@dataclass
class Person:
    name: str
    age: int
    job: str = "Unknown"  # Default value

p = Person("Alice", 30, "Engineer")
print(p)  # Person(name='Alice', age=30, job='Engineer')
print(p == Person("Alice", 30, "Engineer"))  # True

# With additional features
@dataclass(order=True, frozen=False)
class Product:
    name: str
    price: float = field(compare=True)  # Used in comparisons
    quantity: int = field(default=0, compare=False)  # Not compared
    _internal: str = field(default="", repr=False)  # Not in repr

prod1 = Product("Apple", 1.50, 10)
prod2 = Product("Banana", 0.75, 5)
print(prod1 > prod2)  # True (compares price)

# frozen=True makes instances immutable
@dataclass(frozen=True)
class ImmutablePoint:
    x: int
    y: int

point = ImmutablePoint(3, 4)
# point.x = 5  # FrozenInstanceError!

# post_init for additional initialization
@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False)  # Not in __init__
    
    def __post_init__(self):
        self.area = self.width * self.height

rect = Rectangle(5, 3)
print(rect.area)  # 15
```

#### 6.6.3 Descriptors

```python
# Descriptor protocol: __get__, __set__, __delete__
class PositiveNumber:
    def __init__(self, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name, 0)
    
    def __set__(self, obj, value):
        if value < 0:
            raise ValueError(f"{self.name} must be positive")
        obj.__dict__[self.name] = value
    
    def __delete__(self, obj):
        del obj.__dict__[self.name]

class BankAccount:
    balance = PositiveNumber('balance')  # Descriptor instance
    
    def __init__(self, balance):
        self.balance = balance

account = BankAccount(100)
print(account.balance)  # 100

account.balance = 200
print(account.balance)  # 200

# account.balance = -50  # ValueError: balance must be positive

# Properties are actually descriptors!
```

#### 6.6.4 Metaclasses

```python
# Metaclass: class that creates classes
# type is the default metaclass

# Creating class using type()
Person = type('Person', (), {
    '__init__': lambda self, name: setattr(self, 'name', name),
    'greet': lambda self: f"Hello, I'm {self.name}"
})

p = Person("Alice")
print(p.greet())  # Hello, I'm Alice

# Custom metaclass
class SingletonMeta(type):
    """Metaclass that creates Singleton classes"""
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        print("Creating database connection")

db1 = Database()  # Creating database connection
db2 = Database()  # No output (returns existing instance)
print(db1 is db2)  # True

# Metaclasses are powerful but complex; use sparingly!
```

#### 6.6.5 Composition vs Inheritance

```python
# Inheritance: "is-a" relationship
class Vehicle:
    def __init__(self, brand):
        self.brand = brand

class Car(Vehicle):  # Car IS-A Vehicle
    pass

# Composition: "has-a" relationship (often preferred)
class Engine:
    def start(self):
        return "Engine started"
    
    def stop(self):
        return "Engine stopped"

class Car:  # Car HAS-AN Engine
    def __init__(self, brand):
        self.brand = brand
        self.engine = Engine()  # Composition
    
    def start(self):
        return f"{self.brand}: {self.engine.start()}"
    
    def stop(self):
        return f"{self.brand}: {self.engine.stop()}"

car = Car("Toyota")
print(car.start())  # Toyota: Engine started

# Prefer composition when:
# - Relationship is "has-a" not "is-a"
# - Need flexibility to change behavior at runtime
# - Avoid deep inheritance hierarchies
```

---

## 7. ERROR HANDLING

### 7.1 Exception Handling

#### 7.1.1 Basic try/except

```python
# Basic exception handling
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")
# Cannot divide by zero!

# Without except, program crashes
# result = 10 / 0  # ZeroDivisionError: division by zero

# Catching exception and getting details
try:
    number = int("abc")
except ValueError as e:
    print(f"Error: {e}")
# Error: invalid literal for int() with base 10: 'abc'

# Multiple except clauses
try:
    value = int(input("Enter number: "))
    result = 100 / value
except ValueError:
    print("Invalid input!")
except ZeroDivisionError:
    print("Cannot divide by zero!")
```

#### 7.1.2 try/except/else/finally

```python
# else: runs if no exception
# finally: always runs (cleanup code)

def divide(a, b):
    try:
        result = a / b
    except ZeroDivisionError:
        print("Division by zero!")
        return None
    except TypeError:
        print("Invalid types!")
        return None
    else:
        print("Division successful")
        return result
    finally:
        print("Cleanup complete")

print(divide(10, 2))
# Division successful
# Cleanup complete
# 5.0

print(divide(10, 0))
# Division by zero!
# Cleanup complete
# None

# File handling with try/finally
file = None
try:
    file = open('data.txt', 'r')
    data = file.read()
except FileNotFoundError:
    print("File not found!")
finally:
    if file:
        file.close()  # Always close file
    print("File handling complete")
```

#### 7.1.3 Multiple Exceptions

```python
# Catch multiple exception types
try:
    value = int(input("Enter number: "))
    result = 100 / value
    items = [1, 2, 3]
    print(items[value])
except (ValueError, TypeError):  # Tuple of exceptions
    print("Invalid input!")
except ZeroDivisionError:
    print("Cannot divide by zero!")
except IndexError:
    print("Index out of range!")

# Catch all exceptions (use sparingly!)
try:
    risky_operation()
except Exception as e:
    print(f"An error occurred: {e}")

# Bare except (discouraged - catches everything including KeyboardInterrupt)
try:
    risky_operation()
except:
    print("Something went wrong!")  # BAD PRACTICE!
```

#### 7.1.4 Exception Hierarchy

```python
# All exceptions inherit from BaseException
"""
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── StopIteration
    ├── ArithmeticError
    │   ├── FloatingPointError
    │   ├── OverflowError
    │   └── ZeroDivisionError
    ├── AssertionError
    ├── AttributeError
    ├── BufferError
    ├── EOFError
    ├── ImportError
    │   └── ModuleNotFoundError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── MemoryError
    ├── NameError
    │   └── UnboundLocalError
    ├── OSError
    │   ├── FileExistsError
    │   ├── FileNotFoundError
    │   └── PermissionError
    ├── RuntimeError
    │   ├── NotImplementedError
    │   └── RecursionError
    ├── SyntaxError
    │   └── IndentationError
    ├── TypeError
    ├── ValueError
    └── ... many more
"""

# Order matters! Catch specific exceptions first
try:
    risky_operation()
except FileNotFoundError:  # Specific
    print("File not found")
except OSError:  # More general
    print("OS error")
except Exception:  # Most general
    print("Other error")

# Wrong order (never catches OSError!)
# try:
#     risky_operation()
# except Exception:  # Catches everything!
#     print("Error")
# except OSError:  # Never reached!
#     print("OS error")
```

### 7.2 Raising Exceptions

```python
# Raise exception
def validate_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age seems invalid")
    return age

# validate_age(-5)  # ValueError: Age cannot be negative

# Re-raise exception (inside except block)
try:
    validate_age(-5)
except ValueError as e:
    print(f"Caught error: {e}")
    raise  # Re-raises the same exception

# Raise different exception
try:
    data = {"name": "Alice"}
    age = data["age"]
except KeyError:
    raise ValueError("Missing age field") from None

# Exception chaining
try:
    value = int("abc")
except ValueError as e:
    raise RuntimeError("Failed to process data") from e
# Shows both exceptions with "caused by" message
```

### 7.3 Custom Exceptions

```python
# Create custom exception class
class ValidationError(Exception):
    """Custom exception for validation errors"""
    pass

class InsufficientFundsError(Exception):
    """Raised when account has insufficient funds"""
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        message = f"Insufficient funds: balance={balance}, required={amount}"
        super().__init__(message)

class BankAccount:
    def __init__(self, balance):
        self.balance = balance
    
    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError(self.balance, amount)
        self.balance -= amount

account = BankAccount(100)
try:
    account.withdraw(150)
except InsufficientFundsError as e:
    print(f"Error: {e}")
    print(f"Balance: {e.balance}, Attempted: {e.amount}")

# Exception hierarchy for custom exceptions
class APIError(Exception):
    """Base class for API exceptions"""
    pass

class APIConnectionError(APIError):
    """API connection failed"""
    pass

class APITimeoutError(APIError):
    """API request timed out"""
    pass

try:
    api_call()
except APIConnectionError:
    print("Connection failed")
except APITimeoutError:
    print("Request timed out")
except APIError:
    print("API error")
```

### 7.4 Assert Statement

```python
# assert condition, message
# Raises AssertionError if condition is False

def calculate_average(numbers):
    assert len(numbers) > 0, "List cannot be empty"
    return sum(numbers) / len(numbers)

print(calculate_average([1, 2, 3, 4]))  # 2.5
# calculate_average([])  # AssertionError: List cannot be empty

# Assertions can be disabled with -O flag
# python -O script.py  # Disables all assertions

# Use assertions for:
# - Internal self-checks
# - Debugging
# - Documenting assumptions

# DON'T use assertions for:
# - Input validation (use proper exceptions)
# - Production error handling
# - Side effects (assertions can be disabled)

# GOOD
def process_user(user):
    assert isinstance(user, dict), "Internal error: user must be dict"
    # ... process user

# BAD
def withdraw(amount):
    assert amount > 0, "Amount must be positive"  # Use if + raise instead!
```

### 7.5 Context Manager for Cleanup

```python
from contextlib import suppress, contextmanager

# suppress: ignore specific exceptions
with suppress(FileNotFoundError):
    os.remove('file.txt')  # No error if file doesn't exist

# Equivalent to:
try:
    os.remove('file.txt')
except FileNotFoundError:
    pass

# contextmanager decorator
@contextmanager
def temporary_value(var, temp_value):
    """Temporarily change a variable's value"""
    original = var
    try:
        yield temp_value  # Value during 'with' block
    finally:
        var = original  # Restore original

# Multiple context managers
with open('input.txt') as infile, open('output.txt', 'w') as outfile:
    outfile.write(infile.read())
```

---

## 8. FILE OPERATIONS

### 8.1 Basic File I/O

```python
# open() function - modes:
# 'r' - read (default)
# 'w' - write (overwrites existing)
# 'a' - append
# 'x' - exclusive creation (fails if exists)
# 'b' - binary mode
# 't' - text mode (default)
# '+' - read and write

# Reading files
file = open('data.txt', 'r')
content = file.read()  # Read entire file
file.close()

# Better: use 'with' statement (auto-closes)
with open('data.txt', 'r') as file:
    content = file.read()
# File automatically closed

# Reading methods
with open('data.txt', 'r') as file:
    # read() - entire file as string
    content = file.read()
    
    # read(size) - specified number of characters
    chunk = file.read(100)
    
    # readline() - single line
    line = file.readline()
    
    # readlines() - list of all lines
    lines = file.readlines()
    
    # Iterate over file (memory efficient)
    for line in file:
        print(line.strip())

# Writing files
with open('output.txt', 'w') as file:
    file.write("Hello, World!\n")
    file.write("Second line\n")

# Append to file
with open('output.txt', 'a') as file:
    file.write("Appended line\n")

# Write multiple lines
lines = ["Line 1\n", "Line 2\n", "Line 3\n"]
with open('output.txt', 'w') as file:
    file.writelines(lines)

# File object methods
with open('data.txt', 'r+') as file:  # Read and write
    print(file.tell())  # Current position
    file.seek(0)        # Move to beginning
    file.seek(0, 2)     # Move to end (0 offset from end)
    file.flush()        # Force write buffered data
    # file.close()      # Explicit close (not needed with 'with')
```

### 8.2 File Paths

#### 8.2.1 os.path Module

```python
import os

# Join paths (OS-independent)
path = os.path.join('folder', 'subfolder', 'file.txt')
print(path)  # folder/subfolder/file.txt (or folder\\subfolder\\file.txt on Windows)

# Check if path exists
if os.path.exists('data.txt'):
    print("File exists")

# Check if file
if os.path.isfile('data.txt'):
    print("Is a file")

# Check if directory
if os.path.isdir('folder'):
    print("Is a directory")

# Get filename from path
filename = os.path.basename('/path/to/file.txt')
print(filename)  # file.txt

# Get directory from path
dirname = os.path.dirname('/path/to/file.txt')
print(dirname)  # /path/to

# Get absolute path
abspath = os.path.abspath('file.txt')
print(abspath)  # /full/path/to/file.txt

# Split extension
name, ext = os.path.splitext('file.txt')
print(name, ext)  # file .txt

# Get file size
size = os.path.getsize('file.txt')
print(f"Size: {size} bytes")
```

#### 8.2.2 pathlib Module (Modern Approach)

```python
from pathlib import Path

# Create Path object
p = Path('folder') / 'subfolder' / 'file.txt'  # / operator joins paths
print(p)  # folder/subfolder/file.txt

# Check if exists
if p.exists():
    print("Path exists")

# Check if file/directory
print(p.is_file())  # False (doesn't exist yet)
print(p.is_dir())   # False

# Read file (text)
content = Path('data.txt').read_text()
print(content)

# Write file (text)
Path('output.txt').write_text("Hello, World!")

# Read/write bytes
binary_data = Path('image.png').read_bytes()
Path('copy.png').write_bytes(binary_data)

# Create directory
Path('new_folder').mkdir(exist_ok=True)  # No error if exists
Path('nested/folders').mkdir(parents=True, exist_ok=True)  # Create parent dirs

# List directory contents
for item in Path('.').iterdir():
    print(item)

# Glob patterns
for txt_file in Path('.').glob('*.txt'):
    print(txt_file)

# Recursive glob
for py_file in Path('.').rglob('*.py'):  # All .py files recursively
    print(py_file)

# Path properties
p = Path('/home/user/file.txt')
print(p.name)       # file.txt
print(p.stem)       # file
print(p.suffix)     # .txt
print(p.parent)     # /home/user
print(p.absolute()) # Absolute path

# Change extension
new_path = p.with_suffix('.md')
print(new_path)  # /home/user/file.md
```

### 8.3 Binary Files

```python
# Reading binary files
with open('image.png', 'rb') as file:  # 'rb' = read binary
    data = file.read()
    print(type(data))  # <class 'bytes'>

# Writing binary files
data = b'\x89PNG\r\n\x1a\n'  # Binary data
with open('test.bin', 'wb') as file:
    file.write(data)

# struct module for binary data
import struct

# Pack data into binary
data = struct.pack('i f', 42, 3.14)  # int and float
with open('data.bin', 'wb') as file:
    file.write(data)

# Unpack binary data
with open('data.bin', 'rb') as file:
    data = file.read()
    values = struct.unpack('i f', data)
    print(values)  # (42, 3.14)
```

### 8.4 Specific File Formats

#### 8.4.1 CSV Files

```python
import csv

# Writing CSV
data = [
    ['Name', 'Age', 'City'],
    ['Alice', 30, 'NYC'],
    ['Bob', 25, 'LA'],
    ['Charlie', 35, 'Chicago']
]

with open('people.csv', 'w', newline='') as file:
    writer = csv.writer(file)
    writer.writerows(data)

# Reading CSV
with open('people.csv', 'r') as file:
    reader = csv.reader(file)
    for row in reader:
        print(row)  # ['Alice', '30', 'NYC']

# DictReader (reads as dictionaries)
with open('people.csv', 'r') as file:
    reader = csv.DictReader(file)
    for row in reader:
        print(row)  # {'Name': 'Alice', 'Age': '30', 'City': 'NYC'}
        print(row['Name'], row['Age'])

# DictWriter (writes from dictionaries)
data = [
    {'Name': 'Alice', 'Age': 30, 'City': 'NYC'},
    {'Name': 'Bob', 'Age': 25, 'City': 'LA'}
]

with open('people.csv', 'w', newline='') as file:
    fieldnames = ['Name', 'Age', 'City']
    writer = csv.DictWriter(file, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(data)
```

#### 8.4.2 JSON Files

```python
import json

# Python object to JSON string
data = {
    'name': 'Alice',
    'age': 30,
    'skills': ['Python', 'JavaScript'],
    'active': True
}

json_string = json.dumps(data)
print(json_string)  # {"name": "Alice", "age": 30, ...}

# Pretty printing
json_string = json.dumps(data, indent=4)
print(json_string)

# Write JSON to file
with open('data.json', 'w') as file:
    json.dump(data, file, indent=4)

# Read JSON from file
with open('data.json', 'r') as file:
    loaded_data = json.load(file)
    print(loaded_data)

# JSON string to Python object
json_string = '{"name": "Bob", "age": 25}'
data = json.loads(json_string)
print(data['name'])  # Bob

# Handle non-serializable objects
from datetime import datetime

class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

data = {'timestamp': datetime.now()}
json_string = json.dumps(data, cls=DateTimeEncoder)
print(json_string)
```

#### 8.4.3 Pickle (Python Serialization)

```python
import pickle

# Serialize object to file
data = {'name': 'Alice', 'scores': [90, 85, 88]}
with open('data.pkl', 'wb') as file:
    pickle.dump(data, file)

# Deserialize from file
with open('data.pkl', 'rb') as file:
    loaded_data = pickle.load(file)
    print(loaded_data)

# WARNING: Only unpickle data from trusted sources!
# Pickle can execute arbitrary code
```

---

## 9. COMPREHENSIONS

### 9.1 List Comprehensions (Extended)

```python
# Basic
squares = [x**2 for x in range(10)]
print(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# With condition
evens = [x for x in range(20) if x % 2 == 0]
print(evens)  # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# With if-else (ternary)
labels = ['even' if x % 2 == 0 else 'odd' for x in range(10)]
print(labels)  # ['even', 'odd', 'even', ...]

# Nested comprehensions (matrix)
matrix = [[i+j for j in range(3)] for i in range(3)]
print(matrix)  # [[0, 1, 2], [1, 2, 3], [2, 3, 4]]

# Multiple for clauses (cartesian product)
combinations = [(x, y) for x in [1, 2, 3] for y in ['a', 'b']]
print(combinations)  # [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b'), (3, 'a'), (3, 'b')]

# Flattening nested list
nested = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for sublist in nested for num in sublist]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# With function calls
words = ['hello', 'world', 'python']
upper_words = [word.upper() for word in words]
print(upper_words)  # ['HELLO', 'WORLD', 'PYTHON']

# Filter and transform
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
even_squares = [x**2 for x in numbers if x % 2 == 0]
print(even_squares)  # [4, 16, 36, 64, 100]

# Complex conditions
data = [1, -2, 3, -4, 5, -6, 7, -8]
positive_evens = [x for x in data if x > 0 if x % 2 == 0]
print(positive_evens)  # [] (no positive evens in this list)

# Nested with condition
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
filtered = [[num for num in row if num % 2 == 0] for row in matrix]
print(filtered)  # [[2], [4, 6], [8]]
```

### 9.2 Dictionary Comprehensions (Extended)

```python
# Basic
squares = {x: x**2 for x in range(5)}
print(squares)  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# With condition
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}
print(even_squares)  # {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}

# Swap keys and values
original = {'a': 1, 'b': 2, 'c': 3}
swapped = {v: k for k, v in original.items()}
print(swapped)  # {1: 'a', 2: 'b', 3: 'c'}

# From two lists (zip)
keys = ['name', 'age', 'job']
values = ['Alice', 30, 'Engineer']
person = {k: v for k, v in zip(keys, values)}
print(person)  # {'name': 'Alice', 'age': 30, 'job': 'Engineer'}

# Filter dictionary
scores = {'Alice': 85, 'Bob': 72, 'Charlie': 90, 'David': 65}
passed = {name: score for name, score in scores.items() if score >= 75}
print(passed)  # {'Alice': 85, 'Charlie': 90}

# Transform values
prices = {'apple': 1.50, 'banana': 0.75, 'cherry': 2.00}
discounted = {item: price * 0.9 for item, price in prices.items()}
print(discounted)  # {'apple': 1.35, 'banana': 0.675, 'cherry': 1.8}

# Conditional values
numbers = range(1, 6)
categories = {x: 'even' if x % 2 == 0 else 'odd' for x in numbers}
print(categories)  # {1: 'odd', 2: 'even', 3: 'odd', 4: 'even', 5: 'odd'}

# Nested dictionary comprehension
matrix = {i: {j: i*j for j in range(3)} for i in range(3)}
print(matrix)  # {0: {0: 0, 1: 0, 2: 0}, 1: {0: 0, 1: 1, 2: 2}, ...}
```

### 9.3 Set Comprehensions (Extended)

```python
# Basic
squares = {x**2 for x in range(10)}
print(squares)  # {0, 1, 4, 9, 16, 25, 36, 49, 64, 81}

# With condition
even_squares = {x**2 for x in range(10) if x % 2 == 0}
print(even_squares)  # {0, 4, 16, 36, 64}

# Remove duplicates
numbers = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
unique = {x for x in numbers}
print(unique)  # {1, 2, 3, 4}

# String manipulation
words = ['Hello', 'World', 'Python', 'hello', 'WORLD']
lowercase_unique = {word.lower() for word in words}
print(lowercase_unique)  # {'hello', 'world', 'python'}

# Filter and transform
data = ['apple', 'Banana', 'cherry', 'DATE']
capitalized = {word.capitalize() for word in data if len(word) > 4}
print(capitalized)  # {'Apple', 'Banana', 'Cherry'}
```

### 9.4 Generator Expressions (Extended)

```python
# Generator expression (lazy evaluation)
gen = (x**2 for x in range(10))
print(type(gen))  # <class 'generator'>
print(next(gen))  # 0
print(next(gen))  # 1
print(list(gen))  # [4, 9, 16, 25, 36, 49, 64, 81] (rest of values)

# Memory efficient (doesn't create list in memory)
import sys
list_comp = [x**2 for x in range(10000)]  # Creates list
gen_expr = (x**2 for x in range(10000))    # Doesn't create list
print(sys.getsizeof(list_comp))  # Large
print(sys.getsizeof(gen_expr))   # Small (generator object)

# Use in functions that accept iterables
total = sum(x**2 for x in range(100))  # No list created
print(total)

max_val = max(x for x in range(100) if x % 7 == 0)
print(max_val)  # 98

any_negative = any(x < 0 for x in [1, 2, 3, -4, 5])
print(any_negative)  # True

all_positive = all(x > 0 for x in [1, 2, 3, 4, 5])
print(all_positive)  # True

# Chaining generators
gen1 = (x for x in range(5))
gen2 = (y**2 for y in gen1)
print(list(gen2))  # [0, 1, 4, 9, 16]

# When to use generator expressions:
# - Large datasets (memory efficient)
# - One-time iteration
# - Feeding into functions like sum(), max(), any(), all()

# When to use list comprehensions:
# - Need to iterate multiple times
# - Need indexing or slicing
# - Want to see all values immediately
```

---

## 10. BUILT-IN FUNCTIONS

### 10.1 Type Conversion

```python
# int() - convert to integer
print(int("42"))      # 42
print(int(3.14))      # 3 (truncates)
print(int("1010", 2)) # 10 (binary to int)
print(int("FF", 16))  # 255 (hex to int)

# float() - convert to float
print(float("3.14"))  # 3.14
print(float(42))      # 42.0
print(float("inf"))   # inf

# str() - convert to string
print(str(42))        # "42"
print(str([1, 2, 3])) # "[1, 2, 3]"

# bool() - convert to boolean
print(bool(1))        # True
print(bool(0))        # False
print(bool(""))       # False
print(bool("text"))   # True

# list(), tuple(), set(), dict() - convert to respective types
print(list("hello"))           # ['h', 'e', 'l', 'l', 'o']
print(tuple([1, 2, 3]))        # (1, 2, 3)
print(set([1, 2, 2, 3]))       # {1, 2, 3}
print(dict([('a', 1), ('b', 2)]))  # {'a': 1, 'b': 2}
```

### 10.2 Iteration & Sequences

```python
# range() - sequence of numbers
list(range(5))           # [0, 1, 2, 3, 4]
list(range(2, 8))        # [2, 3, 4, 5, 6, 7]
list(range(0, 10, 2))    # [0, 2, 4, 6, 8]
list(range(10, 0, -1))   # [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]

# enumerate() - index and value pairs
fruits = ['apple', 'banana', 'cherry']
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
# 0: apple, 1: banana, 2: cherry

for i, fruit in enumerate(fruits, start=1):
    print(f"{i}: {fruit}")  # Starts from 1

# zip() - parallel iteration
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name} is {age}")

# zip stops at shortest sequence
list(zip([1, 2, 3], ['a', 'b']))  # [(1, 'a'), (2, 'b')]

# strict=True raises exception if lengths differ (Python 3.10+)
# list(zip([1, 2], ['a'], strict=True))  # ValueError

# reversed() - reverse sequence
list(reversed([1, 2, 3]))     # [3, 2, 1]
list(reversed("hello"))       # ['o', 'l', 'l', 'e', 'h']

# sorted() - return sorted list
sorted([3, 1, 4, 1, 5])       # [1, 1, 3, 4, 5]
sorted([3, 1, 4], reverse=True)  # [4, 3, 1]
sorted(['banana', 'apple', 'cherry'])  # ['apple', 'banana', 'cherry']
sorted(['banana', 'apple'], key=len)  # ['apple', 'banana'] (by length)
sorted(['banana', 'Apple'], key=str.lower)  # ['Apple', 'banana'] (case-insensitive)

# iter() - create iterator
it = iter([1, 2, 3])
print(next(it))  # 1
print(next(it))  # 2

# iter() with sentinel value
with open('data.txt', 'r') as f:
    for line in iter(f.readline, ''):  # Read until empty string
        print(line)

# next() - get next item from iterator
it = iter([1, 2, 3])
print(next(it))        # 1
print(next(it, 999))   # 2
print(next(it))        # 3
print(next(it, 999))   # 999 (default, no StopIteration)
```

### 10.3 Functional Programming

```python
# map() - apply function to each item
numbers = [1, 2, 3, 4, 5]
squared = map(lambda x: x**2, numbers)
print(list(squared))  # [1, 4, 9, 16, 25]

# Multiple iterables
list1 = [1, 2, 3]
list2 = [4, 5, 6]
sums = map(lambda x, y: x + y, list1, list2)
print(list(sums))  # [5, 7, 9]

# filter() - filter items by condition
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens = filter(lambda x: x % 2 == 0, numbers)
print(list(evens))  # [2, 4, 6, 8, 10]

# reduce() - reduce to single value (from functools)
from functools import reduce

numbers = [1, 2, 3, 4, 5]
product = reduce(lambda x, y: x * y, numbers)
print(product)  # 120 (1*2*3*4*5)

# With initial value
product = reduce(lambda x, y: x * y, numbers, 10)
print(product)  # 1200 (10*1*2*3*4*5)

# all() - True if all elements are truthy
print(all([True, True, True]))   # True
print(all([True, False, True]))  # False
print(all([1, 2, 3]))            # True
print(all([1, 0, 3]))            # False
print(all([]))                   # True (empty)

# any() - True if any element is truthy
print(any([False, False, False]))  # False
print(any([False, True, False]))   # True
print(any([0, 0, 0]))              # False
print(any([0, 1, 0]))              # True
print(any([]))                     # False (empty)
```

### 10.4 Math & Numbers

```python
# abs() - absolute value
print(abs(-5))      # 5
print(abs(3.14))    # 3.14
print(abs(3+4j))    # 5.0 (magnitude of complex number)

# round() - round to nearest integer
print(round(3.7))       # 4
print(round(3.5))       # 4 (banker's rounding: rounds to even)
print(round(2.5))       # 2
print(round(3.14159, 2))  # 3.14 (2 decimal places)

# pow() - power
print(pow(2, 3))      # 8
print(pow(2, 3, 5))   # 3 (2^3 % 5)

# divmod() - quotient and remainder
print(divmod(17, 5))  # (3, 2)

# sum() - sum of iterable
print(sum([1, 2, 3, 4, 5]))     # 15
print(sum([1, 2, 3], 10))       # 16 (start value)
print(sum([[1, 2], [3, 4]], []))  # [1, 2, 3, 4] (concatenate lists)

# min() / max() - minimum / maximum
print(min([3, 1, 4, 1, 5]))     # 1
print(max([3, 1, 4, 1, 5]))     # 5
print(min('abc', 'xyz', 'def')) # 'abc'
print(max([3, 1, 4], key=lambda x: -x))  # 1 (max by negative value)

# min/max with multiple arguments
print(min(3, 1, 4, 1, 5))  # 1
print(max(3, 1, 4, 1, 5))  # 5
```

### 10.5 String & Character

```python
# chr() - Unicode code point to character
print(chr(65))       # 'A'
print(chr(0x1F600))  # 😀 (emoji)

# ord() - character to Unicode code point
print(ord('A'))      # 65
print(ord('😀'))     # 128512

# format() - format value
print(format(42, 'b'))      # '101010' (binary)
print(format(42, 'x'))      # '2a' (hex)
print(format(3.14159, '.2f'))  # '3.14' (2 decimal places)
print(format(1000000, ','))    # '1,000,000' (comma separator)
```

### 10.6 Object & Type

```python
# type() - get type of object
print(type(42))           # <class 'int'>
print(type("hello"))      # <class 'str'>
print(type([1, 2, 3]))    # <class 'list'>

# isinstance() - check instance of class
print(isinstance(42, int))              # True
print(isinstance(42, (int, str)))       # True (checks multiple)
print(isinstance("hello", str))         # True
print(isinstance([1, 2], (list, tuple))) # True

# issubclass() - check subclass relationship
class Parent:
    pass
class Child(Parent):
    pass

print(issubclass(Child, Parent))  # True
print(issubclass(Parent, Child))  # False
print(issubclass(bool, int))      # True (bool is subclass of int!)

# dir() - list attributes and methods
print(dir([]))  # Lists all list methods

# vars() - __dict__ of object
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

p = Person("Alice", 30)
print(vars(p))  # {'name': 'Alice', 'age': 30}

# help() - interactive help
# help(list)  # Shows documentation for list

# id() - memory address
x = [1, 2, 3]
print(id(x))  # 140234567890 (example)

# hash() - hash value
print(hash("hello"))  # -1234567890 (example)
print(hash(42))       # 42
# print(hash([1, 2]))  # TypeError: unhashable type: 'list'

# callable() - check if object is callable
print(callable(print))    # True
print(callable(lambda x: x))  # True
print(callable(42))       # False
print(callable(list))     # True (can call to create list)
```

### 10.7 I/O

```python
# print() - output to console
print("Hello", "World")  # Hello World
print("Hello", "World", sep=", ")  # Hello, World
print("Hello", end=" ")  # Hello (no newline)
print("World")  # World

import sys
print("Error!", file=sys.stderr)  # Print to stderr

# input() - get user input
# name = input("Enter your name: ")
# print(f"Hello, {name}!")

# open() - open file
with open('file.txt', 'r') as f:
    content = f.read()
```

### 10.8 Number Base Conversions

```python
# bin() - convert to binary string
print(bin(10))   # '0b1010'
print(bin(255))  # '0b11111111'

# hex() - convert to hexadecimal string
print(hex(255))  # '0xff'
print(hex(16))   # '0x10'

# oct() - convert to octal string
print(oct(8))    # '0o10'
print(oct(64))   # '0o100'

# Convert back
print(int('0b1010', 2))   # 10 (binary to int)
print(int('0xff', 16))    # 255 (hex to int)
print(int('0o10', 8))     # 8 (octal to int)
```

### 10.9 Other Important

```python
# len() - length of sequence
print(len([1, 2, 3]))      # 3
print(len("hello"))        # 5
print(len({'a': 1, 'b': 2}))  # 2

# slice() - create slice object
s = slice(1, 5, 2)
print([0, 1, 2, 3, 4, 5][s])  # [1, 3]

# getattr() - get attribute
class Person:
    name = "Alice"

print(getattr(Person, 'name'))  # Alice
print(getattr(Person, 'age', 30))  # 30 (default)

# setattr() - set attribute
setattr(Person, 'age', 25)
print(Person.age)  # 25

# delattr() - delete attribute
delattr(Person, 'age')
# print(Person.age)  # AttributeError

# hasattr() - check if attribute exists
print(hasattr(Person, 'name'))  # True
print(hasattr(Person, 'age'))   # False
```

---

## 11. MODULES & PACKAGES

### 11.1 Importing

```python
# Import entire module
import math
print(math.sqrt(16))  # 4.0

# Import with alias
import math as m
print(m.pi)  # 3.141592653589793

# Import specific items
from math import sqrt, pi
print(sqrt(16))  # 4.0
print(pi)  # 3.141592653589793

# Import with alias
from math import sqrt as square_root
print(square_root(25))  # 5.0

# Import all (avoid in production!)
# from math import *
# print(sqrt(16))  # Works but pollutes namespace

# Relative imports (in packages)
# from . import module         # Same directory
# from .. import module        # Parent directory
# from .subpackage import mod  # Subpackage
```

### 11.2 Module Structure

```python
# mymodule.py
"""
This is my module.

It does awesome things.
"""

__version__ = '1.0.0'
__author__ = 'Your Name'
__all__ = ['public_function', 'PublicClass']  # Controls 'from module import *'

def public_function():
    """This will be imported with *"""
    pass

def _private_function():
    """This won't be imported with * (convention)"""
    pass

class PublicClass:
    """This will be imported with *"""
    pass

# Module-level code (runs on import)
print("Module loaded!")

# Main guard
if __name__ == "__main__":
    # Code here only runs when module is executed directly
    # Not when imported
    print("Running as main script")
    public_function()
else:
    # Code here runs on import
    print("Imported as module")
```

### 11.3 Package Structure

```
mypackage/
├── __init__.py          # Makes directory a package
├── module1.py
├── module2.py
└── subpackage/
    ├── __init__.py
    ├── module3.py
    └── module4.py
```

```python
# mypackage/__init__.py
"""MyPackage - A sample Python package"""

__version__ = '1.0.0'
__all__ = ['module1', 'module2']  # Control what 'from package import *' imports

# Can import submodules here to make them available at package level
from . import module1
from . import module2

# Package initialization code
print("Package initialized")

# Using the package
import mypackage
mypackage.module1.function()

# Or
from mypackage import module1
module1.function()

# Or
from mypackage.module1 import function
function()
```

### 11.4 Module Search Path

```python
import sys

# Python looks for modules in these paths (in order)
print(sys.path)
# [
#     '',  # Current directory
#     '/usr/lib/python3.10',  # Standard library
#     '/usr/local/lib/python3.10/site-packages',  # Third-party packages
#     ...
# ]

# Add custom path
sys.path.append('/custom/path')

# PYTHONPATH environment variable also adds to sys.path
# export PYTHONPATH=/custom/path:$PYTHONPATH
```

### 11.5 Reloading Modules

```python
import importlib
import mymodule

# Module is cached after first import
# Changes to module file won't be reflected

# Reload module
importlib.reload(mymodule)
# Now changes are reflected
```

---

## 12. STANDARD LIBRARY (Essential Modules)

### 12.1 os & sys

```python
import os
import sys

# os module - operating system interface
print(os.getcwd())  # Current working directory
os.chdir('/tmp')   # Change directory
os.listdir('.')    # List directory contents
os.mkdir('newdir') # Create directory
os.remove('file.txt')  # Delete file
os.rename('old.txt', 'new.txt')  # Rename file

# Environment variables
print(os.environ['HOME'])  # Get environment variable
os.environ['MY_VAR'] = 'value'  # Set environment variable

# Path operations (also see os.path section)
print(os.path.join('dir', 'file.txt'))

# sys module - system-specific parameters
print(sys.argv)  # Command-line arguments
print(sys.version)  # Python version
print(sys.platform)  # Platform (linux, win32, darwin, etc.)
print(sys.path)  # Module search path

sys.exit(0)  # Exit program
sys.stdout.write("Hello\n")  # Write to stdout
sys.stderr.write("Error\n")  # Write to stderr
```

### 12.2 datetime

```python
from datetime import datetime, date, time, timedelta, timezone

# Current date and time
now = datetime.now()
print(now)  # 2025-12-05 12:30:45.123456

# Current date
today = date.today()
print(today)  # 2025-12-05

# Create specific datetime
dt = datetime(2025, 12, 25, 10, 30, 0)
print(dt)  # 2025-12-25 10:30:00

# Access components
print(dt.year)    # 2025
print(dt.month)   # 12
print(dt.day)     # 25
print(dt.hour)    # 10
print(dt.minute)  # 30

# Format datetime (strftime)
formatted = dt.strftime("%Y-%m-%d %H:%M:%S")
print(formatted)  # 2025-12-25 10:30:00

# Common format codes:
# %Y - Year (4 digits), %m - Month, %d - Day
# %H - Hour (24h), %I - Hour (12h), %M - Minute, %S - Second
# %A - Weekday name, %B - Month name

# Parse string to datetime (strptime)
date_string = "2025-12-25 10:30:00"
parsed = datetime.strptime(date_string, "%Y-%m-%d %H:%M:%S")
print(parsed)

# timedelta - time difference
delta = timedelta(days=7, hours=2, minutes=30)
future = now + delta
print(future)

diff = datetime(2025, 12, 31) - datetime(2025, 1, 1)
print(diff.days)  # 364

# Timezone-aware datetime
utc_now = datetime.now(timezone.utc)
print(utc_now)
```

### 12.3 math & random

```python
import math
import random

# math module
print(math.pi)    # 3.141592653589793
print(math.e)     # 2.718281828459045
print(math.sqrt(16))  # 4.0
print(math.pow(2, 3)) # 8.0
print(math.ceil(3.2)) # 4 (round up)
print(math.floor(3.8))  # 3 (round down)
print(math.sin(math.pi/2))  # 1.0
print(math.cos(0))    # 1.0
print(math.log(10))   # 2.302585 (natural log)
print(math.log10(100))  # 2.0 (base 10)

# random module
random.seed(42)  # Set seed for reproducibility

# Random float between 0 and 1
print(random.random())  # 0.6394267984578837

# Random integer
print(random.randint(1, 10))  # Random int from 1 to 10 (inclusive)

# Random choice from sequence
colors = ['red', 'green', 'blue']
print(random.choice(colors))  # Random color

# Random sample (multiple unique items)
print(random.sample(range(100), 5))  # 5 unique random numbers

# Shuffle list in place
numbers = [1, 2, 3, 4, 5]
random.shuffle(numbers)
print(numbers)  # Shuffled

# Random float in range
print(random.uniform(1.0, 10.0))  # Random float from 1.0 to 10.0
```

### 12.4 collections

```python
from collections import Counter, defaultdict, namedtuple, deque, ChainMap, OrderedDict

# Counter - count occurrences
words = ['apple', 'banana', 'apple', 'cherry', 'banana', 'apple']
counter = Counter(words)
print(counter)  # Counter({'apple': 3, 'banana': 2, 'cherry': 1})
print(counter.most_common(2))  # [('apple', 3), ('banana', 2)]
print(counter['apple'])  # 3

# defaultdict - dict with default values
dd = defaultdict(int)
dd['a'] += 1
dd['b'] += 2
print(dd)  # defaultdict(<class 'int'>, {'a': 1, 'b': 2})
print(dd['c'])  # 0 (default int value)

# With list
dd_list = defaultdict(list)
dd_list['fruits'].append('apple')
print(dd_list)  # defaultdict(<class 'list'>, {'fruits': ['apple']})

# namedtuple - tuples with named fields
Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x, p.y)  # 10 20
print(p[0], p[1])  # 10 20

# deque - double-ended queue (efficient appends/pops from both ends)
dq = deque([1, 2, 3])
dq.append(4)      # Add to right
dq.appendleft(0)  # Add to left
print(dq)  # deque([0, 1, 2, 3, 4])
print(dq.pop())      # 4 (remove from right)
print(dq.popleft())  # 0 (remove from left)

# Rotation
dq.rotate(1)  # Rotate right
print(dq)

# ChainMap - combine multiple dicts
dict1 = {'a': 1, 'b': 2}
dict2 = {'c': 3, 'd': 4}
chain = ChainMap(dict1, dict2)
print(chain['a'])  # 1
print(chain['c'])  # 3
```

### 12.5 itertools

```python
from itertools import *

# Infinite iterators
counter = count(10, 2)  # Start at 10, step by 2
print(list(islice(counter, 5)))  # [10, 12, 14, 16, 18]

cycler = cycle(['A', 'B', 'C'])
print(list(islice(cycler, 7)))  # ['A', 'B', 'C', 'A', 'B', 'C', 'A']

repeater = repeat('X', 3)
print(list(repeater))  # ['X', 'X', 'X']

# Combining iterables
print(list(chain([1, 2], [3, 4])))  # [1, 2, 3, 4]

# Filtering
print(list(compress('ABCD', [1, 0, 1, 0])))  # ['A', 'C']
print(list(dropwhile(lambda x: x < 5, [1, 4, 6, 4, 1])))  # [6, 4, 1]
print(list(takewhile(lambda x: x < 5, [1, 4, 6, 4, 1])))  # [1, 4]
print(list(filterfalse(lambda x: x % 2 == 0, range(10))))  # [1, 3, 5, 7, 9]

# Slicing
print(list(islice(range(10), 2, 8, 2)))  # [2, 4, 6]

# Combinatorics
print(list(product([1, 2], ['a', 'b'])))  # [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
print(list(permutations([1, 2, 3], 2)))  # [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]
print(list(combinations([1, 2, 3], 2)))  # [(1, 2), (1, 3), (2, 3)]
print(list(combinations_with_replacement([1, 2], 2)))  # [(1, 1), (1, 2), (2, 2)]

# groupby
data = [('A', 1), ('A', 2), ('B', 3), ('B', 4), ('C', 5)]
for key, group in groupby(data, key=lambda x: x[0]):
    print(f"{key}: {list(group)}")
# A: [('A', 1), ('A', 2)]
# B: [('B', 3), ('B', 4)]
# C: [('C', 5)]
```

### 12.6 functools

```python
from functools import reduce, partial, lru_cache, wraps, total_ordering, singledispatch

# reduce - covered earlier
numbers = [1, 2, 3, 4, 5]
product = reduce(lambda x, y: x * y, numbers)
print(product)  # 120

# partial - create function with fixed arguments
def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)
print(square(5))  # 25
print(cube(5))    # 125

# lru_cache - memoization decorator
@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # Fast!
print(fibonacci.cache_info())  # CacheInfo(hits=..., misses=..., maxsize=128, currsize=...)

# wraps - preserve function metadata in decorators
def my_decorator(func):
    @wraps(func)  # Preserves func's name, docstring, etc.
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

# total_ordering - generate comparison methods
@total_ordering
class Student:
    def __init__(self, name, grade):
        self.name = name
        self.grade = grade
    
    def __eq__(self, other):
        return self.grade == other.grade
    
    def __lt__(self, other):
        return self.grade < other.grade
    # Other comparison methods generated automatically!

# singledispatch - function overloading based on type
@singledispatch
def process(arg):
    print(f"Generic: {arg}")

@process.register(int)
def _(arg):
    print(f"Integer: {arg}")

@process.register(str)
def _(arg):
    print(f"String: {arg}")

process(42)      # Integer: 42
process("hello")  # String: hello
process([1, 2])  # Generic: [1, 2]
```

### 12.7 re (Regular Expressions)

```python
import re

# Basic patterns
text = "The rain in Spain falls mainly in the plain."

# search() - find first match
match = re.search(r'Spain', text)
if match:
    print(f"Found: {match.group()}")  # Found: Spain
    print(f"Position: {match.start()}-{match.end()}")  # Position: 12-17

# match() - match at beginning of string
match = re.match(r'The', text)
print(match.group() if match else "No match")  # The

# findall() - find all matches
matches = re.findall(r'in', text)
print(matches)  # ['in', 'in', 'in', 'in', 'in']

# finditer() - iterator of match objects
for match in re.finditer(r'in', text):
    print(f"{match.group()} at {match.start()}")

# sub() - replace matches
new_text = re.sub(r'in', 'ON', text)
print(new_text)  # The raON ON SpaON falls maONly ON the plaON.

# split() - split by pattern
words = re.split(r'\s+', text)  # Split on whitespace
print(words)

# Pattern groups
text = "John: 30, Jane: 25"
matches = re.findall(r'(\w+):\s*(\d+)', text)
print(matches)  # [('John', '30'), ('Jane', '25')]

# Named groups
pattern = r'(?P<name>\w+):\s*(?P<age>\d+)'
for match in re.finditer(pattern, text):
    print(f"{match.group('name')} is {match.group('age')}")

# Compiled patterns (for reuse)
pattern = re.compile(r'\d+')
print(pattern.findall("123 abc 456"))  # ['123', '456']

# Flags
# re.IGNORECASE - case-insensitive
# re.MULTILINE - ^ and $ match line boundaries
# re.DOTALL - . matches newlines
print(re.findall(r'spain', text, re.IGNORECASE))  # ['Spain']

# Common patterns:
# \d - digit, \D - non-digit
# \w - word char (letter, digit, underscore), \W - non-word
# \s - whitespace, \S - non-whitespace
# . - any character (except newline)
# * - 0 or more, + - 1 or more, ? - 0 or 1
# {n} - exactly n, {n,} - n or more, {n,m} - between n and m
# ^ - start of string, $ - end of string
# [] - character class, [^] - negated character class
# () - group, (?:) - non-capturing group
# | - OR
```

### 12.8 argparse

```python
import argparse

# Create parser
parser = argparse.ArgumentParser(description='Process some integers.')

# Positional argument
parser.add_argument('filename', help='input file name')

# Optional argument
parser.add_argument('-v', '--verbose', action='store_true',
                    help='increase output verbosity')

# Optional with value
parser.add_argument('-o', '--output', type=str, default='output.txt',
                    help='output file name')

# Integer argument
parser.add_argument('-n', '--number', type=int, default=10,
                    help='number of items')

# Choices
parser.add_argument('--format', choices=['json', 'xml', 'csv'],
                    default='json', help='output format')

# Multiple values
parser.add_argument('--values', nargs='+', type=int,
                    help='list of values')

# Parse arguments
# args = parser.parse_args()  # From command line
args = parser.parse_args(['input.txt', '-v', '--number', '5'])  # Example

print(args.filename)  # input.txt
print(args.verbose)   # True
print(args.number)    # 5
```

### 12.9 logging

```python
import logging

# Basic configuration
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='app.log'
)

# Log levels (from lowest to highest):
# DEBUG, INFO, WARNING, ERROR, CRITICAL

logging.debug("Debug message")
logging.info("Info message")
logging.warning("Warning message")
logging.error("Error message")
logging.critical("Critical message")

# Create logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# Create handler
handler = logging.FileHandler('app.log')
handler.setLevel(logging.INFO)

# Create formatter
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

# Add handler to logger
logger.addHandler(handler)

logger.info("This is an info message")

# Exception logging
try:
    1 / 0
except Exception:
    logger.exception("An error occurred")  # Includes traceback
```

### 12.10 typing (Type Hints)

```python
from typing import List, Dict, Set, Tuple, Optional, Union, Any, Callable, TypeVar, Generic

# Basic types
def greet(name: str) -> str:
    return f"Hello, {name}"

# Collections
def process_numbers(numbers: List[int]) -> int:
    return sum(numbers)

def get_user() -> Dict[str, Union[str, int]]:
    return {"name": "Alice", "age": 30}

# Optional (can be None)
def find_user(user_id: int) -> Optional[str]:
    if user_id == 1:
        return "Alice"
    return None

# Union (multiple types)
def process(value: Union[int, str]) -> str:
    return str(value)

# Callable (function type)
def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

# TypeVar for generic types
T = TypeVar('T')

def first(items: List[T]) -> T:
    return items[0]

# Generic class
class Stack(Generic[T]):
    def __init__(self) -> None:
        self.items: List[T] = []
    
    def push(self, item: T) -> None:
        self.items.append(item)
    
    def pop(self) -> T:
        return self.items.pop()

# Type aliases
Vector = List[float]
Matrix = List[Vector]

def scale(scalar: float, vector: Vector) -> Vector:
    return [scalar * num for num in vector]
```

---

## 13. ADVANCED CONCEPTS (Extended)

### 13.1 Generators (Extended)

```python
# Generator function
def count_up_to(n):
    i = 1
    while i <= n:
        yield i
        i += 1

for num in count_up_to(5):
    print(num)  # 1, 2, 3, 4, 5

# Generator methods: send(), throw(), close()
def echo_generator():
    while True:
        value = yield
        print(f"Received: {value}")

gen = echo_generator()
next(gen)  # Prime the generator
gen.send("Hello")  # Received: Hello
gen.send("World")  # Received: World
gen.close()  # Stop generator

# Generator pipelines
def read_file(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()

def filter_lines(lines):
    for line in lines:
        if line:
            yield line

def transform_lines(lines):
    for line in lines:
        yield line.upper()

# Chain generators
# lines = read_file('data.txt')
# filtered = filter_lines(lines)
# transformed = transform_lines(filtered)
# for line in transformed:
#     print(line)
```

### 13.2 Iterators

```python
# Iterator protocol: __iter__ and __next__
class CountDown:
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

counter = CountDown(5)
for num in counter:
    print(num)  # 5, 4, 3, 2, 1

# Iterable vs Iterator
# Iterable: has __iter__() method (list, tuple, dict, etc.)
# Iterator: has __iter__() and __next__() methods

my_list = [1, 2, 3]  # Iterable
my_iter = iter(my_list)  # Iterator
print(next(my_iter))  # 1
print(next(my_iter))  # 2
```

### 13.3 Context Managers (Extended)

```python
from contextlib import contextmanager

# Using __enter__ and __exit__
class ManagedFile:
    def __init__(self, filename):
        self.filename = filename
    
    def __enter__(self):
        self.file = open(self.filename, 'w')
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        # Return True to suppress exception, False to propagate
        return False

with ManagedFile('test.txt') as f:
    f.write("Hello, World!")

# Using contextmanager decorator
@contextmanager
def managed_resource(resource_name):
    print(f"Acquiring {resource_name}")
    try:
        yield resource_name
    finally:
        print(f"Releasing {resource_name}")

with managed_resource("database connection") as resource:
    print(f"Using {resource}")
# Acquiring database connection
# Using database connection
# Releasing database connection
```

### 13.4 Asyncio (Basics)

```python
import asyncio

# Basic async function (coroutine)
async def greet(name, delay):
    await asyncio.sleep(delay)
    print(f"Hello, {name}!")
    return f"Greeted {name}"

# Run coroutine
# asyncio.run(greet("Alice", 1))

# Multiple coroutines concurrently
async def main():
    # await greet("Alice", 2)
    # await greet("Bob", 1)  # Sequential (total 3 seconds)
    
    # Run concurrently
    results = await asyncio.gather(
        greet("Alice", 2),
        greet("Bob", 1),
        greet("Charlie", 1.5)
    )  # Total ~2 seconds (concurrent)
    print(results)

# asyncio.run(main())

# Create task
async def task_example():
    task = asyncio.create_task(greet("Alice", 1))
    # Do other work
    await task  # Wait for task to complete

# Common patterns
async def fetch_data(url):
    # Simulating async I/O
    await asyncio.sleep(1)
    return f"Data from {url}"

async def fetch_all(urls):
    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks)
    return results
```

---

## 14. COMMON PATTERNS & IDIOMS

### 14.1 Pythonic Patterns

```python
# EAFP vs LBYL
# LBYL: Look Before You Leap
if 'key' in my_dict:
    value = my_dict['key']
else:
    value = None

# EAFP: Easier to Ask Forgiveness than Permission (Pythonic)
try:
    value = my_dict['key']
except KeyError:
    value = None

# Or better:
value = my_dict.get('key')

# enumerate() instead of range(len())
items = ['a', 'b', 'c']

# Bad
for i in range(len(items)):
    print(f"{i}: {items[i]}")

# Good
for i, item in enumerate(items):
    print(f"{i}: {item}")

# zip() for parallel iteration
names = ['Alice', 'Bob']
ages = [30, 25]

# Bad
for i in range(len(names)):
    print(f"{names[i]} is {ages[i]}")

# Good
for name, age in zip(names, ages):
    print(f"{name} is {age}")

# Dictionary get() with defaults
# Bad
if 'key' in my_dict:
    value = my_dict['key']
else:
    value = 'default'

# Good
value = my_dict.get('key', 'default')

# Any/all for boolean checks
numbers = [1, 2, 3, 4, 5]

# Check if any even
has_even = any(n % 2 == 0 for n in numbers)

# Check if all positive
all_positive = all(n > 0 for n in numbers)

# Unpacking
# Bad
first = items[0]
rest = items[1:]

# Good
first, *rest = items

# Context managers for resources
# Bad
f = open('file.txt')
data = f.read()
f.close()

# Good
with open('file.txt') as f:
    data = f.read()
```

### 14.2 Common Gotchas (Summary)

```python
# 1. Mutable default arguments
def append_to(element, target=[]):  # DON'T DO THIS!
    target.append(element)
    return target

# Fix: Use None
def append_to(element, target=None):
    if target is None:
        target = []
    target.append(element)
    return target

# 2. Late binding in closures
functions = []
for i in range(3):
    functions.append(lambda: i)  # i is bound late!

for f in functions:
    print(f())  # All print 2!

# Fix: Use default argument
functions = []
for i in range(3):
    functions.append(lambda i=i: i)  # i bound immediately

for f in functions:
    print(f())  # Prints 0, 1, 2

# 3. is vs ==
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True (values equal)
print(a is b)  # False (different objects)

# Use 'is' only for None, True, False
if value is None:
    pass

# 4. Shallow vs deep copy
import copy
original = [[1, 2], [3, 4]]
shallow = original.copy()
deep = copy.deepcopy(original)

original[0][0] = 999
print(shallow)  # [[999, 2], [3, 4]] - affected!
print(deep)     # [[1, 2], [3, 4]] - not affected
```

---

## 15. TESTING & DEBUGGING

### 15.1 Debugging with pdb

```python
import pdb

# Set breakpoint
def buggy_function(x, y):
    result = x + y
    pdb.set_trace()  # Debugger stops here
    return result * 2

# Python 3.7+: use breakpoint()
def buggy_function(x, y):
    result = x + y
    breakpoint()  # Stops here
    return result * 2

# Common pdb commands:
# n (next) - execute next line
# s (step) - step into function
# c (continue) - continue execution
# l (list) - show source code
# p variable - print variable
# pp variable - pretty print variable
# q (quit) - exit debugger
# h (help) - show help
```

### 15.2 unittest

```python
import unittest

def add(a, b):
    return a + b

class TestAddFunction(unittest.TestCase):
    def setUp(self):
        """Run before each test"""
        self.x = 10
        self.y = 5
    
    def tearDown(self):
        """Run after each test"""
        pass
    
    def test_add_positive(self):
        self.assertEqual(add(2, 3), 5)
    
    def test_add_negative(self):
        self.assertEqual(add(-1, -1), -2)
    
    def test_add_zero(self):
        self.assertEqual(add(5, 0), 5)
    
    def test_add_floats(self):
        self.assertAlmostEqual(add(0.1, 0.2), 0.3, places=1)

# Run tests
if __name__ == '__main__':
    unittest.main()

# Common assertions:
# assertEqual(a, b)
# assertNotEqual(a, b)
# assertTrue(x)
# assertFalse(x)
# assertIs(a, b)
# assertIsNone(x)
# assertIn(a, b)
# assertIsInstance(a, b)
# assertRaises(Exception)
```

### 15.3 pytest (Brief)

```python
# test_example.py
def add(a, b):
    return a + b

# Test functions (start with test_)
def test_add_positive():
    assert add(2, 3) == 5

def test_add_negative():
    assert add(-1, -1) == -2

# Fixtures
import pytest

@pytest.fixture
def sample_data():
    return [1, 2, 3, 4, 5]

def test_sum(sample_data):
    assert sum(sample_data) == 15

# Parametrize
@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
])
def test_add_parametrized(a, b, expected):
    assert add(a, b) == expected

# Run: pytest test_example.py
```

---

## 16. VIRTUAL ENVIRONMENTS & PACKAGE MANAGEMENT

### 16.1 Virtual Environments

```bash
# Create virtual environment
python -m venv myenv

# Activate (Linux/Mac)
source myenv/bin/activate

# Activate (Windows)
myenv\Scripts\activate

# Deactivate
deactivate

# Why use virtual environments?
# - Isolate project dependencies
# - Different Python versions per project
# - Avoid dependency conflicts
# - Reproducible environments
```

### 16.2 pip

```bash
# Install package
pip install requests

# Install specific version
pip install requests==2.28.0

# Install minimum version
pip install requests>=2.28.0

# Upgrade package
pip install --upgrade requests

# Uninstall package
pip uninstall requests

# List installed packages
pip list

# Show package info
pip show requests

# Freeze dependencies
pip freeze > requirements.txt

# Install from requirements file
pip install -r requirements.txt

# Install in development mode
pip install -e .

# Search for package
pip search package_name  # (deprecated)
```

---

## 17. BEST PRACTICES & STYLE

### 17.1 PEP 8 Style Guide

```python
# Naming conventions
class MyClass:  # CamelCase for classes
    pass

def my_function():  # lowercase_with_underscores for functions
    pass

my_variable = 42  # lowercase_with_underscores for variables
MY_CONSTANT = 100  # UPPERCASE_WITH_UNDERSCORES for constants

# Indentation: 4 spaces (not tabs)
def example():
    if True:
        print("Correct indentation")

# Line length: max 79 characters (or 88 with black)
long_variable_name = (
    "This is a long string that "
    "spans multiple lines"
)

# Imports
# 1. Standard library
import os
import sys

# 2. Third-party
import requests
import numpy as np

# 3. Local
from mypackage import mymodule

# Whitespace
# Correct
x = 1
y = 2
z = x + y

# Wrong
x=1
y= 2
z =x+y

# Function/method spacing
class Example:
    def method1(self):
        pass
    
    def method2(self):  # One blank line between methods
        pass

# Two blank lines before top-level classes/functions
class AnotherClass:
    pass


def another_function():
    pass
```

### 17.2 Documentation

```python
# Module docstring
"""
This module provides utility functions for data processing.

Functions:
    process_data: Process raw data
    validate_input: Validate user input
"""

def function_example(param1, param2):
    """
    Brief description of function.
    
    Longer description if needed. Explain what the function does,
    its parameters, return value, and any exceptions.
    
    Args:
        param1 (int): Description of param1
        param2 (str): Description of param2
    
    Returns:
        bool: Description of return value
    
    Raises:
        ValueError: When input is invalid
        TypeError: When wrong type is passed
    
    Example:
        >>> function_example(5, "test")
        True
    """
    pass

class ExampleClass:
    """
    Brief description of class.
    
    Longer description if needed.
    
    Attributes:
        attr1 (str): Description of attribute
        attr2 (int): Description of attribute
    """
    pass
```

---

## QUICK REFERENCE

### Common Operations One-Liners

```python
# String operations
reversed_str = text[::-1]
uppercase = text.upper()
words = text.split()
joined = ' '.join(words)

# List operations
reversed_list = lst[::-1]
filtered = [x for x in lst if x > 0]
sorted_list = sorted(lst)
flattened = [item for sublist in nested for item in sublist]

# Dictionary operations
keys = list(my_dict.keys())
values = list(my_dict.values())
merged = {**dict1, **dict2}
filtered_dict = {k: v for k, v in my_dict.items() if v > 0}

# Set operations
union = set1 | set2
intersection = set1 & set2
difference = set1 - set2

# File operations
content = Path('file.txt').read_text()
Path('file.txt').write_text(content)
lines = Path('file.txt').read_text().splitlines()

# Functional
sum_all = sum(numbers)
max_val = max(numbers)
filtered = filter(lambda x: x > 0, numbers)
mapped = map(lambda x: x**2, numbers)
```

### Operator Precedence (High to Low)

```
1. ()                    Parentheses
2. **                    Exponentiation
3. +x, -x, ~x           Unary
4. *, /, //, %          Multiplication, Division
5. +, -                  Addition, Subtraction
6. <<, >>               Bitwise shifts
7. &                     Bitwise AND
8. ^                     Bitwise XOR
9. |                     Bitwise OR
10. ==, !=, <, >, <=, >= Comparisons
11. is, is not           Identity
12. in, not in           Membership
13. not                  Boolean NOT
14. and                  Boolean AND
15. or                   Boolean OR
```

### String Formatting Quick Reference

```python
# f-strings (preferred)
f"{value}"
f"{value:10}"        # Width 10
f"{value:<10}"       # Left align
f"{value:>10}"       # Right align
f"{value:^10}"       # Center
f"{value:05d}"       # Zero padding
f"{value:.2f}"       # 2 decimal places
f"{value:,.2f}"      # With comma separator
f"{value:.2%}"       # Percentage
f"{value:b}"         # Binary
f"{value:x}"         # Hexadecimal

# Format method
"{0} {1}".format(a, b)
"{name} {age}".format(name="Alice", age=30)

# % formatting (old style)
"%s %d" % (string, number)
"%.2f" % float_num
```

### Essential Built-in Functions

```python
# Type conversion
int(), float(), str(), bool(), list(), tuple(), set(), dict()

# Iteration
range(), enumerate(), zip(), reversed(), sorted()
iter(), next()

# Functional
map(), filter(), reduce()  # reduce from functools
all(), any(), sum(), min(), max()

# Object/Type
type(), isinstance(), issubclass()
dir(), vars(), help()
id(), hash(), callable()

# I/O
print(), input(), open()

# String/Character
chr(), ord(), format()

# Math
abs(), round(), pow(), divmod()

# Number base
bin(), hex(), oct()

# Attributes
getattr(), setattr(), delattr(), hasattr()

# Other
len(), slice()
```

### Important Module Imports

```python
# File operations
from pathlib import Path
import os, os.path
import shutil

# Data structures
from collections import Counter, defaultdict, namedtuple, deque
from collections import OrderedDict, ChainMap

# Iteration tools
from itertools import *  # chain, combinations, permutations, etc.

# Functional tools
from functools import reduce, partial, lru_cache, wraps

# Time/Date
from datetime import datetime, date, time, timedelta, timezone
import time

# Math
import math
import random
import statistics

# String/Regex
import re
import string

# System
import sys
import argparse
import logging

# Data formats
import json
import csv
import pickle

# Type hints
from typing import List, Dict, Set, Tuple, Optional, Union, Any
from typing import Callable, TypeVar, Generic

# Testing
import unittest
import pytest  # Third-party

# Async
import asyncio
```

### Time Complexity (Big O) Reference

```python
# List
lst[i]           # O(1) - indexing
lst.append(x)    # O(1) - append
lst.insert(0, x) # O(n) - insert at beginning
lst.pop()        # O(1) - pop from end
lst.pop(0)       # O(n) - pop from beginning
x in lst         # O(n) - membership

# Dictionary
dict[key]        # O(1) - access
dict[key] = val  # O(1) - set
del dict[key]    # O(1) - delete
key in dict      # O(1) - membership

# Set
x in s           # O(1) - membership
s.add(x)         # O(1) - add
s.remove(x)      # O(1) - remove
s | s2           # O(len(s) + len(s2)) - union
s & s2           # O(min(len(s), len(s2))) - intersection

# Common algorithms
sorted(lst)      # O(n log n) - sorting
min(lst), max(lst)  # O(n) - find min/max
```

---

**END OF CHEATSHEET**

This comprehensive Python cheatsheet covers everything from basic syntax to advanced concepts. Use it as a quick reference guide for your Python programming needs!
