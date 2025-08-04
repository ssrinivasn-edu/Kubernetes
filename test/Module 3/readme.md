
## Module 3: Modules, Packages, and Imports (2 Hours)

### Theory: Python Module System

A module is a file containing Python code. Packages are directories containing multiple modules. This system allows code organization and reusability.

#### Types of Imports

```python
# Import entire module
import math
result = math.sqrt(16)

# Import specific functions
from math import sqrt, pi
result = sqrt(16)

# Import with alias
import numpy as np
array = np.array([1, 2, 3])

# Import all (not recommended)
from math import *
result = sqrt(16)
```

### Built-in Modules

#### OS Module

```python
import os

# Environment variables
home_dir = os.environ.get('HOME')
path = os.environ.get('PATH')

# Path operations
current_dir = os.getcwd()
parent_dir = os.path.dirname(current_dir)
full_path = os.path.join(current_dir, 'subfolder', 'file.txt')

# File system operations
files = os.listdir('.')
os.makedirs('new_directory', exist_ok=True)
```

#### Datetime Module

```python
from datetime import datetime, timedelta, date

# Current date and time
now = datetime.now()
today = date.today()

# Formatting
formatted = now.strftime('%Y-%m-%d %H:%M:%S')
parsed = datetime.strptime('2023-12-25', '%Y-%m-%d')

# Arithmetic
tomorrow = today + timedelta(days=1)
last_week = now - timedelta(weeks=1)
```

#### Collections Module

```python
from collections import defaultdict, Counter, namedtuple

# defaultdict
dd = defaultdict(list)
dd['key'].append('value')  # No KeyError

# Counter
text = "hello world"
letter_count = Counter(text)
print(letter_count.most_common(3))

# namedtuple
Person = namedtuple('Person', ['name', 'age', 'city'])
john = Person('John', 30, 'New York')
print(john.name)
```

### Creating Custom Modules

#### mathutils.py
```python
"""
Custom math utilities module
"""

def factorial(n):
    """Calculate factorial of a number"""
    if n < 0:
        raise ValueError("Factorial is not defined for negative numbers")
    if n == 0 or n == 1:
        return 1
    return n * factorial(n - 1)

def is_prime(n):
    """Check if a number is prime"""
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

def fibonacci(n):
    """Generate fibonacci sequence up to n terms"""
    if n <= 0:
        return []
    elif n == 1:
        return [0]
    elif n == 2:
        return [0, 1]
    
    fib = [0, 1]
    for i in range(2, n):
        fib.append(fib[i-1] + fib[i-2])
    return fib

# Module-level variables
PI = 3.14159265359
E = 2.71828182846

# Module initialization code
print(f"Mathutils module loaded. PI = {PI}")
```

#### Using the custom module

```python
# Import the entire module
import mathutils

print(mathutils.factorial(5))
print(mathutils.is_prime(17))
print(mathutils.fibonacci(10))
print(mathutils.PI)

# Import specific functions
from mathutils import factorial, is_prime

print(factorial(5))
print(is_prime(17))

# Import with alias
import mathutils as mu

print(mu.fibonacci(8))
```

### Creating Packages

#### Package Structure
```
mypackage/
    __init__.py
    math/
        __init__.py
        basic.py
        advanced.py
    string/
        __init__.py
        processing.py
        validation.py
```

#### mypackage/__init__.py
```python
"""
MyPackage - A collection of utility functions
Version: 1.0.0
"""

__version__ = "1.0.0"
__author__ = "Your Name"

# Import commonly used functions to package level
from .math.basic import add, subtract, multiply, divide
from .string.processing import clean_text, extract_numbers

# Package-level function
def get_version():
    return __version__

# What gets imported with "from mypackage import *"
__all__ = ['add', 'subtract', 'multiply', 'divide', 'clean_text', 'extract_numbers']
```

#### mypackage/math/basic.py
```python
"""Basic mathematical operations"""

def add(a, b):
    """Add two numbers"""
    return a + b

def subtract(a, b):
    """Subtract two numbers"""
    return a - b

def multiply(a, b):
    """Multiply two numbers"""
    return a * b

def divide(a, b):
    """Divide two numbers"""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

def power(base, exponent):
    """Calculate power of a number"""
    return base ** exponent
```

#### mypackage/string/processing.py
```python
"""String processing utilities"""

import re

def clean_text(text):
    """Clean text by removing extra whitespace and special characters"""
    # Remove extra whitespace
    text = ' '.join(text.split())
    # Remove special characters except spaces
    text = re.sub(r'[^a-zA-Z0-9\s]', '', text)
    return text.strip()

def extract_numbers(text):
    """Extract all numbers from text"""
    return re.findall(r'\d+', text)

def word_count(text):
    """Count words in text"""
    return len(text.split())

def capitalize_words(text):
    """Capitalize each word in text"""
    return ' '.join(word.capitalize() for word in text.split())
```

#### Using the package

```python
# Import from package
from mypackage import add, clean_text
from mypackage.math.basic import power
from mypackage.string.processing import word_count

# Use imported functions
result = add(5, 3)
cleaned = clean_text("  Hello,   World!  ")
word_total = word_count("This is a test sentence")

print(f"Addition result: {result}")
print(f"Cleaned text: '{cleaned}'")
print(f"Word count: {word_total}")
```

### Advanced Package Features

#### Relative Imports

```python
# mypackage/math/advanced.py
from .basic import add, multiply  # Relative import from same package
from ..string.processing import clean_text  # Import from sibling package

def compound_interest(principal, rate, time):
    """Calculate compound interest"""
    amount = multiply(principal, power(add(1, rate), time))
    return amount - principal
```

#### Conditional Imports

```python
# Optional dependency handling
try:
    import numpy as np
    HAS_NUMPY = True
except ImportError:
    HAS_NUMPY = False

def advanced_calculation(data):
    if HAS_NUMPY:
        return np.mean(data)
    else:
        return sum(data) / len(data)
```

### Hands-on Exercise 3.1: File Processing Package

```python
# fileprocessor/__init__.py
"""
File Processing Package
Handles various file operations and formats
"""

__version__ = "1.0.0"

from .readers import TextReader, CSVReader, JSONReader
from .writers import TextWriter, CSVWriter, JSONWriter
from .processors import DataProcessor

class FileManager:
    """Main interface for file operations"""
    
    def __init__(self):
        self.readers = {
            'txt': TextReader(),
            'csv': CSVReader(),
            'json': JSONReader()
        }
        self.writers = {
            'txt': TextWriter(),
            'csv': CSVWriter(),
            'json': JSONWriter()
        }
        self.processor = DataProcessor()
    
    def read_file(self, filename):
        """Read file based on extension"""
        extension = filename.split('.')[-1].lower()
        if extension in self.readers:
            return self.readers[extension].read(filename)
        else:
            raise ValueError(f"Unsupported file type: {extension}")
    
    def write_file(self, filename, data):
        """Write file based on extension"""
        extension = filename.split('.')[-1].lower()
        if extension in self.writers:
            return self.writers[extension].write(filename, data)
        else:
            raise ValueError(f"Unsupported file type: {extension}")

# fileprocessor/readers.py
import csv
import json

class BaseReader:
    """Base class for file readers"""
    def read(self, filename):
        raise NotImplementedError

class TextReader(BaseReader):
    def read(self, filename):
        with open(filename, 'r', encoding='utf-8') as file:
            return file.read()

class CSVReader(BaseReader):
    def read(self, filename):
        data = []
        with open(filename, 'r', newline='') as csvfile:
            reader = csv.DictReader(csvfile)
            for row in reader:
                data.append(row)
        return data

class JSONReader(BaseReader):
    def read(self, filename):
        with open(filename, 'r') as jsonfile:
            return json.load(jsonfile)

# Usage
from fileprocessor import FileManager

fm = FileManager()
data = fm.read_file('data.csv')
fm.write_file('output.json', data)
```

---
