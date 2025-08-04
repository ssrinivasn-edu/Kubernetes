# 5-Day Python Programming Training
## Master Automation, Testing, and Professional Development

---

# Day 1: Foundation & File Operations

## Module 1: File Handling & Automation Scripts (3 Hours)

### Theory: File I/O Operations

Python provides built-in functions and modules for file operations. The core concept revolves around opening files, performing operations, and properly closing them.

#### Basic File Operations

```python
# Opening files with different modes
# 'r' - read, 'w' - write, 'a' - append, 'x' - create
file = open('filename.txt', 'r')
content = file.read()
file.close()

# Using context manager (recommended)
with open('filename.txt', 'r') as file:
    content = file.read()
    # File automatically closed when exiting the block
```

#### File Modes and Their Usage

| Mode | Description | Creates New File | Overwrites |
|------|-------------|------------------|------------|
| 'r' | Read only | No | N/A |
| 'w' | Write only | Yes | Yes |
| 'a' | Append | Yes | No |
| 'r+' | Read and Write | No | No |
| 'w+' | Write and Read | Yes | Yes |

### Practical Examples

#### Example 1: Reading Different File Types

```python
# Reading text files
def read_text_file(filename):
    try:
        with open(filename, 'r', encoding='utf-8') as file:
            return file.read()
    except FileNotFoundError:
        print(f"File {filename} not found")
        return None

# Reading CSV files
import csv

def read_csv_file(filename):
    data = []
    with open(filename, 'r', newline='') as csvfile:
        reader = csv.reader(csvfile)
        for row in reader:
            data.append(row)
    return data

# Reading JSON files
import json

def read_json_file(filename):
    with open(filename, 'r') as jsonfile:
        return json.load(jsonfile)

# Example usage
text_content = read_text_file('sample.txt')
csv_data = read_csv_file('data.csv')
json_data = read_json_file('config.json')
```

#### Example 2: Writing Files

```python
import csv
import json

def write_text_file(filename, content):
    with open(filename, 'w', encoding='utf-8') as file:
        file.write(content)

def write_csv_file(filename, data):
    with open(filename, 'w', newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerows(data)

def write_json_file(filename, data):
    with open(filename, 'w') as jsonfile:
        json.dump(data, jsonfile, indent=4)

# Example usage
write_text_file('output.txt', 'Hello, World!')
write_csv_file('output.csv', [['Name', 'Age'], ['John', 30], ['Jane', 25]])
write_json_file('output.json', {'name': 'John', 'age': 30})
```

### File System Automation

#### Working with OS Module

```python
import os
import shutil
from pathlib import Path

# Get current working directory
current_dir = os.getcwd()
print(f"Current directory: {current_dir}")

# List files in directory
def list_files(directory):
    return [f for f in os.listdir(directory) if os.path.isfile(os.path.join(directory, f))]

# Create directory
def create_directory(dir_name):
    os.makedirs(dir_name, exist_ok=True)

# Check if file/directory exists
def check_exists(path):
    return os.path.exists(path)

# Get file information
def get_file_info(filename):
    stat = os.stat(filename)
    return {
        'size': stat.st_size,
        'modified': stat.st_mtime,
        'created': stat.st_ctime
    }
```

#### File Operations with Shutil

```python
import shutil
import os

def copy_file(src, dst):
    """Copy file from source to destination"""
    shutil.copy2(src, dst)

def move_file(src, dst):
    """Move file from source to destination"""
    shutil.move(src, dst)

def delete_file(filename):
    """Delete a file"""
    if os.path.exists(filename):
        os.remove(filename)
        return True
    return False

def rename_file(old_name, new_name):
    """Rename a file"""
    os.rename(old_name, new_name)

def copy_directory(src, dst):
    """Copy entire directory"""
    shutil.copytree(src, dst)
```

#### Automation Script Example: File Organizer

```python
import os
import shutil
from pathlib import Path

class FileOrganizer:
    def __init__(self, source_dir):
        self.source_dir = Path(source_dir)
        self.file_types = {
            'images': ['.jpg', '.jpeg', '.png', '.gif', '.bmp'],
            'documents': ['.pdf', '.doc', '.docx', '.txt', '.rtf'],
            'videos': ['.mp4', '.avi', '.mkv', '.mov', '.wmv'],
            'audio': ['.mp3', '.wav', '.flac', '.aac'],
            'archives': ['.zip', '.rar', '.7z', '.tar', '.gz']
        }
    
    def organize_files(self):
        """Organize files by type into subdirectories"""
        if not self.source_dir.exists():
            print(f"Directory {self.source_dir} does not exist")
            return
        
        # Create directories for each file type
        for category in self.file_types:
            category_dir = self.source_dir / category
            category_dir.mkdir(exist_ok=True)
        
        # Move files to appropriate directories
        for file_path in self.source_dir.iterdir():
            if file_path.is_file():
                file_extension = file_path.suffix.lower()
                moved = False
                
                for category, extensions in self.file_types.items():
                    if file_extension in extensions:
                        destination = self.source_dir / category / file_path.name
                        shutil.move(str(file_path), str(destination))
                        print(f"Moved {file_path.name} to {category}/")
                        moved = True
                        break
                
                if not moved:
                    # Create 'others' directory for unknown file types
                    others_dir = self.source_dir / 'others'
                    others_dir.mkdir(exist_ok=True)
                    destination = others_dir / file_path.name
                    shutil.move(str(file_path), str(destination))
                    print(f"Moved {file_path.name} to others/")

# Usage
organizer = FileOrganizer('/path/to/messy/directory')
organizer.organize_files()
```

### Hands-on Exercise 1.1: Log File Analyzer

```python
import re
from datetime import datetime
from collections import defaultdict

class LogAnalyzer:
    def __init__(self, log_file):
        self.log_file = log_file
        self.log_data = []
        self.error_count = defaultdict(int)
        self.ip_addresses = defaultdict(int)
    
    def parse_log_file(self):
        """Parse Apache/Nginx log file"""
        log_pattern = r'(\d+\.\d+\.\d+\.\d+) - - \[(.*?)\] "(.*?)" (\d+) (\d+)'
        
        with open(self.log_file, 'r') as file:
            for line in file:
                match = re.match(log_pattern, line)
                if match:
                    ip, timestamp, request, status, size = match.groups()
                    self.log_data.append({
                        'ip': ip,
                        'timestamp': timestamp,
                        'request': request,
                        'status': int(status),
                        'size': int(size)
                    })
                    
                    # Count IP addresses
                    self.ip_addresses[ip] += 1
                    
                    # Count error codes
                    if int(status) >= 400:
                        self.error_count[status] += 1
    
    def generate_report(self, output_file):
        """Generate analysis report"""
        with open(output_file, 'w') as report:
            report.write("Log Analysis Report\n")
            report.write("==================\n\n")
            
            report.write(f"Total requests: {len(self.log_data)}\n\n")
            
            report.write("Top 10 IP Addresses:\n")
            for ip, count in sorted(self.ip_addresses.items(), 
                                  key=lambda x: x[1], reverse=True)[:10]:
                report.write(f"{ip}: {count} requests\n")
            
            report.write("\nError Status Codes:\n")
            for status, count in self.error_count.items():
                report.write(f"HTTP {status}: {count} errors\n")

# Usage
analyzer = LogAnalyzer('access.log')
analyzer.parse_log_file()
analyzer.generate_report('log_analysis_report.txt')
```

---

## Module 2: Exception and Error Handling (3 Hours)

### Theory: Understanding Exceptions

Exceptions are runtime errors that occur during program execution. Python uses a try-except mechanism to handle these errors gracefully.

#### Types of Errors

1. **Syntax Errors**: Detected before program execution
2. **Runtime Errors**: Occur during program execution
3. **Logical Errors**: Program runs but produces incorrect results

#### Common Built-in Exceptions

```python
# FileNotFoundError
try:
    with open('nonexistent.txt', 'r') as file:
        content = file.read()
except FileNotFoundError:
    print("File not found!")

# ValueError
try:
    number = int("not_a_number")
except ValueError:
    print("Invalid value for conversion!")

# ZeroDivisionError
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")

# IndexError
try:
    my_list = [1, 2, 3]
    print(my_list[10])
except IndexError:
    print("Index out of range!")
```

### Exception Handling Blocks

#### Basic Try-Except Structure

```python
try:
    # Code that might raise an exception
    risky_operation()
except SpecificException:
    # Handle specific exception
    handle_specific_error()
except AnotherException as e:
    # Handle another exception with error details
    print(f"Error occurred: {e}")
except Exception as e:
    # Catch all other exceptions
    print(f"Unexpected error: {e}")
else:
    # Executed if no exception occurs
    print("Operation completed successfully")
finally:
    # Always executed, regardless of exceptions
    cleanup_resources()
```

#### Multiple Exception Handling

```python
def safe_division(a, b):
    try:
        result = a / b
        return result
    except ZeroDivisionError:
        print("Error: Division by zero")
        return None
    except TypeError:
        print("Error: Invalid types for division")
        return None
    except Exception as e:
        print(f"Unexpected error: {e}")
        return None

# Usage examples
print(safe_division(10, 2))    # 5.0
print(safe_division(10, 0))    # Error: Division by zero
print(safe_division(10, "2"))  # Error: Invalid types for division
```

### Custom Exceptions

```python
class CustomError(Exception):
    """Base class for custom exceptions"""
    pass

class ValidationError(CustomError):
    """Raised when input validation fails"""
    def __init__(self, message, field_name=None):
        self.message = message
        self.field_name = field_name
        super().__init__(self.message)

class DatabaseConnectionError(CustomError):
    """Raised when database connection fails"""
    def __init__(self, message, error_code=None):
        self.message = message
        self.error_code = error_code
        super().__init__(self.message)

# Using custom exceptions
def validate_email(email):
    if '@' not in email:
        raise ValidationError("Invalid email format", "email")
    if len(email) < 5:
        raise ValidationError("Email too short", "email")
    return True

try:
    validate_email("invalid-email")
except ValidationError as e:
    print(f"Validation failed for {e.field_name}: {e.message}")
```

### Best Practices for Error Handling

#### Example: Robust File Processing

```python
import json
import logging

# Configure logging
logging.basicConfig(level=logging.INFO, 
                   format='%(asctime)s - %(levelname)s - %(message)s')

class FileProcessor:
    def __init__(self):
        self.logger = logging.getLogger(__name__)
    
    def process_json_file(self, filename):
        """Process JSON file with comprehensive error handling"""
        try:
            # Check if file exists
            if not os.path.exists(filename):
                raise FileNotFoundError(f"File {filename} not found")
            
            # Open and read file
            with open(filename, 'r', encoding='utf-8') as file:
                data = json.load(file)
            
            # Validate data structure
            if not isinstance(data, dict):
                raise ValueError("JSON data must be a dictionary")
            
            # Process data
            processed_data = self._process_data(data)
            
            self.logger.info(f"Successfully processed {filename}")
            return processed_data
            
        except FileNotFoundError as e:
            self.logger.error(f"File error: {e}")
            return None
        except json.JSONDecodeError as e:
            self.logger.error(f"JSON parsing error: {e}")
            return None
        except ValueError as e:
            self.logger.error(f"Data validation error: {e}")
            return None
        except PermissionError:
            self.logger.error(f"Permission denied accessing {filename}")
            return None
        except Exception as e:
            self.logger.error(f"Unexpected error processing {filename}: {e}")
            return None
    
    def _process_data(self, data):
        """Private method to process the data"""
        # Implement your data processing logic here
        processed = {}
        for key, value in data.items():
            if isinstance(value, str):
                processed[key] = value.upper()
            else:
                processed[key] = value
        return processed

# Usage
processor = FileProcessor()
result = processor.process_json_file('config.json')
if result:
    print("File processed successfully:", result)
```

### Hands-on Exercise 2.1: Database Connection Manager

```python
import sqlite3
import logging
from contextlib import contextmanager

class DatabaseError(Exception):
    """Custom database exception"""
    pass

class DatabaseManager:
    def __init__(self, db_path):
        self.db_path = db_path
        self.logger = logging.getLogger(__name__)
    
    @contextmanager
    def get_connection(self):
        """Context manager for database connections"""
        conn = None
        try:
            conn = sqlite3.connect(self.db_path)
            conn.row_factory = sqlite3.Row  # Enable column access by name
            yield conn
            conn.commit()
        except sqlite3.Error as e:
            if conn:
                conn.rollback()
            self.logger.error(f"Database error: {e}")
            raise DatabaseError(f"Database operation failed: {e}")
        finally:
            if conn:
                conn.close()
    
    def execute_query(self, query, params=None):
        """Execute a query with error handling"""
        try:
            with self.get_connection() as conn:
                cursor = conn.cursor()
                if params:
                    cursor.execute(query, params)
                else:
                    cursor.execute(query)
                
                if query.strip().upper().startswith('SELECT'):
                    return cursor.fetchall()
                else:
                    return cursor.rowcount
        except DatabaseError:
            raise
        except Exception as e:
            self.logger.error(f"Query execution failed: {e}")
            raise DatabaseError(f"Query execution failed: {e}")
    
    def create_table(self, table_name, columns):
        """Create table with error handling"""
        try:
            query = f"CREATE TABLE IF NOT EXISTS {table_name} ({columns})"
            self.execute_query(query)
            self.logger.info(f"Table {table_name} created successfully")
        except DatabaseError as e:
            self.logger.error(f"Failed to create table {table_name}: {e}")
            raise

# Usage example
try:
    db = DatabaseManager('example.db')
    db.create_table('users', 'id INTEGER PRIMARY KEY, name TEXT, email TEXT')
    
    # Insert data
    db.execute_query("INSERT INTO users (name, email) VALUES (?, ?)", 
                    ("John Doe", "john@example.com"))
    
    # Query data
    users = db.execute_query("SELECT * FROM users")
    for user in users:
        print(f"User: {user['name']}, Email: {user['email']}")
        
except DatabaseError as e:
    print(f"Database operation failed: {e}")
```

---

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

# Day 2: Data Processing & APIs

## Module 4: Regular Expressions for Automation (3 Hours)

### Theory: Pattern Matching with Regex

Regular expressions (regex) are powerful tools for pattern matching and text manipulation. They use special characters to define search patterns.

#### Basic Regex Patterns

```python
import re

# Basic patterns
pattern = r'hello'              # Literal string
pattern = r'[abc]'              # Character class - matches a, b, or c
pattern = r'[a-z]'              # Range - any lowercase letter
pattern = r'[A-Z]'              # Range - any uppercase letter
pattern = r'[0-9]'              # Range - any digit
pattern = r'\d'                 # Digit shorthand
pattern = r'\w'                 # Word character (letter, digit, underscore)
pattern = r'\s'                 # Whitespace character
```

#### Quantifiers

```python
# Quantifiers
pattern = r'a*'                 # Zero or more 'a'
pattern = r'a+'                 # One or more 'a'
pattern = r'a?'                 # Zero or one 'a'
pattern = r'a{3}'               # Exactly 3 'a's
pattern = r'a{2,5}'             # Between 2 and 5 'a's
pattern = r'a{2,}'              # 2 or more 'a's
```

#### Anchors and Boundaries

```python
# Anchors
pattern = r'^hello'             # Start of string
pattern = r'world$'             # End of string
pattern = r'\bword\b'           # Word boundary
```

### Common Regex Operations

#### Finding Patterns

```python
import re

text = "The quick brown fox jumps over the lazy dog"

# Search for first occurrence
match = re.search(r'fox', text)
if match:
    print(f"Found 'fox' at position {match.start()}")

# Find all occurrences
matches = re.findall(r'\b\w{4}\b', text)  # Find all 4-letter words
print(f"4-letter words: {matches}")

# Find with positions
for match in re.finditer(r'\b\w+\b', text):
    print(f"Word: {match.group()}, Position: {match.start()}-{match.end()}")
```

#### Replacing Patterns

```python
import re

text = "Contact us at info@example.com or support@example.com"

# Simple replacement
result = re.sub(r'example\.com', 'newdomain.com', text)
print(result)

# Replacement with groups
result = re.sub(r'(\w+)@example\.com', r'\1@newdomain.com', text)
print(result)

# Replacement with function
def mask_email(match):
    email = match.group(0)
    username, domain = email.split('@')
    masked_username = username[0] + '*' * (len(username) - 2) + username[-1]
    return f"{masked_username}@{domain}"

result = re.sub(r'\b\w+@\w+\.\w+\b', mask_email, text)
print(result)
```

### Practical Regex Examples

#### Email Validation

```python
import re

def validate_email(email):
    """Validate email address using regex"""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

# Test cases
emails = [
    "user@example.com",
    "user.name@example.com",
    "user+tag@example.com",
    "invalid.email",
    "user@",
    "@example.com"
]

for email in emails:
    if validate_email(email):
        print(f"✓ {email} is valid")
    else:
        print(f"✗ {email} is invalid")
```

#### Phone Number Extraction

```python
import re

def extract_phone_numbers(text):
    """Extract phone numbers from text"""
    patterns = [
        r'\b\d{3}-\d{3}-\d{4}\b',           # 123-456-7890
        r'\b\(\d{3}\)\s*\d{3}-\d{4}\b',     # (123) 456-7890
        r'\b\d{3}\.\d{3}\.\d{4}\b',         # 123.456.7890
        r'\b\d{10}\b',                      # 1234567890
        r'\b\+1\s*\d{3}\s*\d{3}\s*\d{4}\b' # +1 123 456 7890
    ]
    
    phone_numbers = []
    for pattern in patterns:
        matches = re.findall(pattern, text)
        phone_numbers.extend(matches)
    
    return list(set(phone_numbers))  # Remove duplicates

# Test
text = """
Contact information:
- Office: 123-456-7890
- Mobile: (555) 123-4567
- Fax: 555.987.6543
- International: +1 800 555 0123
- Invalid: 12345
"""

phones = extract_phone_numbers(text)
for phone in phones:
    print(f"Found phone: {phone}")
```

#### Data Cleaning with Regex

```python
import re

class TextCleaner:
    """Text cleaning utility using regular expressions"""
    
    def __init__(self):
        self.patterns = {
            'extra_spaces': r'\s+',
            'html_tags': r'<[^>]+>',
            'emails': r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
            'urls': r'https?://(?:[-\w.])+(?:[:\d]+)?(?:/(?:[\w/_.])*(?:\?(?:[\w&=%.])*)?(?:#(?:\w)*)?)?',
            'phone_numbers': r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b',
            'special_chars': r'[^\w\s]',
            'numbers': r'\d+'
        }
    
    def remove_html_tags(self, text):
        """Remove HTML tags from text"""
        return re.sub(self.patterns['html_tags'], '', text)
    
    def normalize_whitespace(self, text):
        """Replace multiple whitespace with single space"""
        return re.sub(self.patterns['extra_spaces'], ' ', text).strip()
    
    def extract_emails(self, text):
        """Extract email addresses from text"""
        return re.findall(self.patterns['emails'], text)
    
    def extract_urls(self, text):
        """Extract URLs from text"""
        return re.findall(self.patterns['urls'], text)
    
    def mask_sensitive_data(self, text):
        """Mask emails and phone numbers"""
        # Mask emails
        text = re.sub(self.patterns['emails'], '[EMAIL]', text)
        # Mask phone numbers
        text = re.sub(self.patterns['phone_numbers'], '[PHONE]', text)
        return text
    
    def clean_text(self, text, remove_numbers=False, remove_special=False):
        """Comprehensive text cleaning"""
        # Remove HTML tags
        text = self.remove_html_tags(text)
        
        # Normalize whitespace
        text = self.normalize_whitespace(text)
        
        # Remove numbers if requested
        if remove_numbers:
            text = re.sub(self.patterns['numbers'], '', text)
        
        # Remove special characters if requested
        if remove_special:
            text = re.sub(self.patterns['special_chars'], ' ', text)
            text = self.normalize_whitespace(text)
        
        return text

# Usage example
cleaner = TextCleaner()

sample_text = """
<p>Contact   us at    info@example.com or call 123-456-7890.</p>
<div>Visit our website: https://www.example.com for more info!</div>
Price: $29.99 (limited time offer)
"""

print("Original text:")
print(sample_text)

print("\nCleaned text:")
cleaned = cleaner.clean_text(sample_text, remove_special=True)
print(cleaned)

print("\nExtracted emails:")
print(cleaner.extract_emails(sample_text))

print("\nExtracted URLs:")
print(cleaner.extract_urls(sample_text))

print("\nMasked sensitive data:")
print(cleaner.mask_sensitive_data(sample_text))
```

### Log File Processing with Regex

```python
import re
from datetime import datetime
from collections import defaultdict, Counter

class LogProcessor:
    """Process log files using regular expressions"""
    
    def __init__(self):
        # Common log patterns
        self.patterns = {
            'apache_common': r'(\S+) \S+ \S+ \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) (\S+)" (\d{3}) (\d+|-)',
            'apache_combined': r'(\S+) \S+ \S+ \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) (\S+)" (\d{3}) (\d+|-) "([^"]*)" "([^"]*)"',
            'nginx': r'(\S+) - - \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) (\S+)" (\d{3}) (\d+) "([^"]*)" "([^"]*)"',
            'error': r'\[([\w\s:]+)\] \[(\w+)\] (.+)',
        }
        
        self.stats = {
            'total_requests': 0,
            'status_codes': Counter(),
            'ip_addresses': Counter(),
            'user_agents': Counter(),
            'error_messages': [],
            'top_pages': Counter()
        }
    
    def parse_apache_log(self, log_line, pattern_type='apache_common'):
        """Parse Apache log line"""
        pattern = self.patterns.get(pattern_type)
        if not pattern:
            return None
        
        match = re.match(pattern, log_line)
        if match:
            groups = match.groups()
            if pattern_type == 'apache_common':
                return {
                    'ip': groups[0],
                    'timestamp': groups[1],
                    'method': groups[2],
                    'url': groups[3],
                    'protocol': groups[4],
                    'status': groups[5],
                    'size': groups[6]
                }
            elif pattern_type == 'apache_combined':
                return {
                    'ip': groups[0],
                    'timestamp': groups[1],
                    'method': groups[2],
                    'url': groups[3],
                    'protocol': groups[4],
                    'status': groups[5],
                    'size': groups[6],
                    'referrer': groups[7],
                    'user_agent': groups[8]
                }
        return None
    
    def process_log_file(self, filename, pattern_type='apache_common'):
        """Process entire log file"""
        processed_lines = []
        
        with open(filename, 'r') as file:
            for line_num, line in enumerate(file, 1):
                parsed = self.parse_apache_log(line.strip(), pattern_type)
                if parsed:
                    processed_lines.append(parsed)
                    self._update_stats(parsed)
                else:
                    print(f"Warning: Could not parse line {line_num}")
        
        return processed_lines
    
    def _update_stats(self, parsed_line):
        """Update statistics"""
        self.stats['total_requests'] += 1
        self.stats['status_codes'][parsed_line['status']] += 1
        self.stats['ip_addresses'][parsed_line['ip']] += 1
        self.stats['top_pages'][parsed_line['url']] += 1
        
        if 'user_agent' in parsed_line:
            self.stats['user_agents'][parsed_line['user_agent']] += 1
    
    def generate_report(self, output_file):
        """Generate analysis report"""
        with open(output_file, 'w') as report:
            report.write("Log Analysis Report\n")
            report.write("=" * 50 + "\n\n")
            
            # Summary statistics
            report.write(f"Total Requests: {self.stats['total_requests']}\n\n")
            
            # Top status codes
            report.write("Status Code Distribution:\n")
            report.write("-" * 30 + "\n")
            for status, count in self.stats['status_codes'].most_common():
                percentage = (count / self.stats['total_requests']) * 100
                report.write(f"{status}: {count} ({percentage:.1f}%)\n")
            
            # Top IP addresses
            report.write("\nTop 10 IP Addresses:\n")
            report.write("-" * 30 + "\n")
            for ip, count in self.stats['ip_addresses'].most_common(10):
                report.write(f"{ip}: {count} requests\n")
            
            # Top pages
            report.write("\nTop 10 Requested Pages:\n")
            report.write("-" * 30 + "\n")
            for page, count in self.stats['top_pages'].most_common(10):
                report.write(f"{page}: {count} requests\n")
    
    def find_suspicious_activity(self):
        """Find suspicious IP addresses (high request volume)"""
        total_requests = self.stats['total_requests']
        threshold = total_requests * 0.05  # 5% of total requests
        
        suspicious_ips = []
        for ip, count in self.stats['ip_addresses'].items():
            if count > threshold:
                suspicious_ips.append((ip, count))
        
        return sorted(suspicious_ips, key=lambda x: x[1], reverse=True)

# Usage example
processor = LogProcessor()

# Create sample log data for testing
sample_log = '''127.0.0.1 - - [10/Oct/2023:13:55:36 +0000] "GET /index.html HTTP/1.1" 200 2326
192.168.1.1 - - [10/Oct/2023:13:55:37 +0000] "POST /login HTTP/1.1" 200 1234
10.0.0.1 - - [10/Oct/2023:13:55:38 +0000] "GET /admin HTTP/1.1" 403 1024
127.0.0.1 - - [10/Oct/2023:13:55:39 +0000] "GET /images/logo.png HTTP/1.1" 200 5432'''

# Save sample log to file
with open('sample.log', 'w') as f:
    f.write(sample_log)

# Process the log file
logs = processor.process_log_file('sample.log')
processor.generate_report('log_analysis.txt')

print("Log processing complete. Check log_analysis.txt for results.")
```

### Hands-on Exercise 4.1: Data Validation System

```python
import re
from typing import Dict, List, Tuple

class DataValidator:
    """Comprehensive data validation using regular expressions"""
    
    def __init__(self):
        self.validation_patterns = {
            'email': r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,},
            'phone_us': r'^\+?1?[-.\s]?\(?([0-9]{3})\)?[-.\s]?([0-9]{3})[-.\s]?([0-9]{4}),
            'ssn': r'^\d{3}-\d{2}-\d{4},
            'credit_card': r'^\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4},
            'ip_address': r'^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?),
            'url': r'^https?:\/\/(?:www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b(?:[-a-zA-Z0-9()@:%_\+.~#?&=]*),
            'password_strong': r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,},
            'date_iso': r'^\d{4}-\d{2}-\d{2},
            'time_24h': r'^([01]?[0-9]|2[0-3]):[0-5][0-9],
            'postal_code_us': r'^\d{5}(-\d{4})?
        }
    
    def validate_field(self, field_type: str, value: str) -> Tuple[bool, str]:
        """Validate a single field"""
        if field_type not in self.validation_patterns:
            return False, f"Unknown field type: {field_type}"
        
        pattern = self.validation_patterns[field_type]
        if re.match(pattern, value):
            return True, "Valid"
        else:
            return False, f"Invalid {field_type} format"
    
    def validate_data(self, data: Dict[str, str]) -> Dict[str, Tuple[bool, str]]:
        """Validate multiple fields"""
        results = {}
        for field_name, (field_type, value) in data.items():
            results[field_name] = self.validate_field(field_type, value)
        return results
    
    def clean_phone_number(self, phone: str) -> str:
        """Clean and format phone number"""
        # Remove all non-digit characters
        digits = re.sub(r'\D', '', phone)
        
        # Handle US phone numbers
        if len(digits) == 10:
            return f"({digits[:3]}) {digits[3:6]}-{digits[6:]}"
        elif len(digits) == 11 and digits[0] == '1':
            return f"+1 ({digits[1:4]}) {digits[4:7]}-{digits[7:]}"
        else:
            return phone  # Return original if can't format
    
    def extract_and_validate_emails(self, text: str) -> List[Tuple[str, bool]]:
        """Extract emails from text and validate them"""
        # Find potential email patterns
        email_pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
        potential_emails = re.findall(email_pattern, text)
        
        validated_emails = []
        for email in potential_emails:
            is_valid, _ = self.validate_field('email', email)
            validated_emails.append((email, is_valid))
        
        return validated_emails
    
    def sanitize_input(self, text: str) -> str:
        """Sanitize user input by removing potentially harmful patterns"""
        # Remove HTML tags
        text = re.sub(r'<[^>]+>', '', text)
        
        # Remove SQL injection patterns
        sql_patterns = [
            r'(\bUNION\b|\bSELECT\b|\bINSERT\b|\bDELETE\b|\bUPDATE\b|\bDROP\b)',
            r'(--|/\*|\*/)',
            r'(\'\s*OR\s*\'|\'\s*AND\s*\')'
        ]
        for pattern in sql_patterns:
            text = re.sub(pattern, '', text, flags=re.IGNORECASE)
        
        # Remove excessive whitespace
        text = re.sub(r'\s+', ' ', text).strip()
        
        return text

# Usage example
validator = DataValidator()

# Test data
test_data = {
    'user_email': ('email', 'user@example.com'),
    'user_phone': ('phone_us', '(555) 123-4567'),
    'user_ssn': ('ssn', '123-45-6789'),
    'website': ('url', 'https://www.example.com'),
    'password': ('password_strong', 'StrongPass123!'),
    'birth_date': ('date_iso', '1990-12-25')
}

# Validate all fields
results = validator.validate_data(test_data)

print("Validation Results:")
print("-" * 40)
for field, (is_valid, message) in results.items():
    status = "✓" if is_valid else "✗"
    print(f"{status} {field}: {message}")

# Test email extraction and validation
sample_text = """
Contact us at info@example.com or support@test.org
Also try admin@invalid or user@domain.
Visit https://www.example.com for more info.
"""

print("\nExtracted and validated emails:")
emails = validator.extract_and_validate_emails(sample_text)
for email, is_valid in emails:
    status = "✓" if is_valid else "✗"
    print(f"{status} {email}")

# Test input sanitization
malicious_input = """
<script>alert('XSS')</script>
'; DROP TABLE users; --
Hello   World   with   extra   spaces
"""

print("\nSanitized input:")
print(f"Original: {repr(malicious_input)}")
print(f"Sanitized: {repr(validator.sanitize_input(malicious_input))}")
```

---

## Module 5: Working with Databases (SQLite) (1 Hour)

### Theory: Database Fundamentals

SQLite is a lightweight, serverless database engine that's perfect for desktop applications and small to medium-sized projects. It stores the entire database in a single file.

#### Basic SQL Operations

```python
import sqlite3
from contextlib import contextmanager

# Basic connection
conn = sqlite3.connect('example.db')
cursor = conn.cursor()

# Create table
cursor.execute('''
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        age INTEGER,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
''')

# Insert data
cursor.execute('''
    INSERT INTO users (name, email, age) 
    VALUES (?, ?, ?)
''', ('John Doe', 'john@example.com', 30))

# Commit and close
conn.commit()
conn.close()
```

### Database Manager Class

```python
import sqlite3
import logging
from datetime import datetime
from typing import List, Dict, Any, Optional

class DatabaseManager:
    """SQLite database manager with error handling and utilities"""
    
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.logger = logging.getLogger(__name__)
        self._init_database()
    
    def _init_database(self):
        """Initialize database with basic configuration"""
        with self.get_connection() as conn:
            # Enable foreign key support
            conn.execute("PRAGMA foreign_keys = ON")
            # Set journal mode for better concurrency
            conn.execute("PRAGMA journal_mode = WAL")
    
    @contextmanager
    def get_connection(self):
        """Context manager for database connections"""
        conn = None
        try:
            conn = sqlite3.connect(self.db_path)
            conn.row_factory = sqlite3.Row  # Enable column access by name
            yield conn
            conn.commit()
        except sqlite3.Error as e:
            if conn:
                conn.rollback()
            self.logger.error(f"Database error: {e}")
            raise
        finally:
            if conn:
                conn.close()
    
    def execute_query(self, query: str, params: tuple = None) -> List[sqlite3.Row]:
        """Execute a query and return results"""
        with self.get_connection() as conn:
            cursor = conn.cursor()
            if params:
                cursor.execute(query, params)
            else:
                cursor.execute(query)
            
            if query.strip().upper().startswith('SELECT'):
                return cursor.fetchall()
            else:
                return cursor.rowcount
    
    def execute_many(self, query: str, data: List[tuple]) -> int:
        """Execute query with multiple parameter sets"""
        with self.get_connection() as conn:
            cursor = conn.cursor()
            cursor.executemany(query, data)
            return cursor.rowcount
    
    def create_table(self, table_name: str, columns: str):
        """Create a table"""
        query = f"CREATE TABLE IF NOT EXISTS {table_name} ({columns})"
        self.execute_query(query)
        self.logger.info(f"Table {table_name} created/verified")
    
    def insert_record(self, table: str, data: Dict[str, Any]) -> int:
        """Insert a record and return the ID"""
        columns = ', '.join(data.keys())
        placeholders = ', '.join(['?' for _ in data])
        query = f"INSERT INTO {table} ({columns}) VALUES ({placeholders})"
        
        with self.get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute(query, tuple(data.values()))
            return cursor.lastrowid
    
    def update_record(self, table: str, data: Dict[str, Any], where: str, params: tuple = None) -> int:
        """Update records"""
        set_clause = ', '.join([f"{col} = ?" for col in data.keys()])
        query = f"UPDATE {table} SET {set_clause} WHERE {where}"
        
        update_params = tuple(data.values())
        if params:
            update_params += params
        
        return self.execute_query(query, update_params)
    
    def delete_record(self, table: str, where: str, params: tuple = None) -> int:
        """Delete records"""
        query = f"DELETE FROM {table} WHERE {where}"
        return self.execute_query(query, params)
    
    def get_table_info(self, table: str) -> List[sqlite3.Row]:
        """Get table schema information"""
        return self.execute_query(f"PRAGMA table_info({table})")
    
    def backup_database(self, backup_path: str):
        """Create a backup of the database"""
        with self.get_connection() as source:
            backup_conn = sqlite3.connect(backup_path)
            source.backup(backup_conn)
            backup_conn.close()
        self.logger.info(f"Database backed up to {backup_path}")

# Example usage
db = DatabaseManager('company.db')

# Create tables
db.create_table('departments', '''
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    budget REAL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
''')

db.create_table('employees', '''
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    department_id INTEGER,
    salary REAL,
    hire_date DATE,
    FOREIGN KEY (department_id) REFERENCES departments (id)
''')
```

### Advanced Database Operations

```python
class EmployeeManager:
    """Manager for employee-related database operations"""
    
    def __init__(self, db_manager: DatabaseManager):
        self.db = db_manager
    
    def add_department(self, name: str, budget: float = 0) -> int:
        """Add a new department"""
        data = {'name': name, 'budget': budget}
        return self.db.insert_record('departments', data)
    
    def add_employee(self, name: str, email: str, department_id: int, 
                    salary: float, hire_date: str = None) -> int:
        """Add a new employee"""
        if hire_date is None:
            hire_date = datetime.now().strftime('%Y-%m-%d')
        
        data = {
            'name': name,
            'email': email,
            'department_id': department_id,
            'salary': salary,
            'hire_date': hire_date
        }
        return self.db.insert_record('employees', data)
    
    def get_employees_by_department(self, department_name: str) -> List[Dict]:
        """Get all employees in a specific department"""
        query = '''
            SELECT e.id, e.name, e.email, e.salary, e.hire_date, d.name as department
            FROM employees e
            JOIN departments d ON e.department_id = d.id
            WHERE d.name = ?
            ORDER BY e.name
        '''
        rows = self.db.execute_query(query, (department_name,))
        return [dict(row) for row in rows]
    
    def get_salary_statistics(self) -> Dict[str, float]:
        """Get salary statistics"""
        query = '''
            SELECT 
                AVG(salary) as avg_salary,
                MIN(salary) as min_salary,
                MAX(salary) as max_salary,
                COUNT(*) as total_employees
            FROM employees
        '''
        result = self.db.execute_query(query)[0]
        return dict(result)
    
    def update_salary(self, employee_id: int, new_salary: float) -> bool:
        """Update employee salary"""
        affected = self.db.update_record(
            'employees', 
            {'salary': new_salary}, 
            'id = ?', 
            (employee_id,)
        )
        return affected > 0
    
    def get_department_budget_usage(self) -> List[Dict]:
        """Calculate budget usage by department"""
        query = '''
            SELECT 
                d.name as department,
                d.budget,
                COALESCE(SUM(e.salary), 0) as total_salaries,
                d.budget - COALESCE(SUM(e.salary), 0) as remaining_budget,
                CASE 
                    WHEN d.budget > 0 THEN 
                        (COALESCE(SUM(e.salary), 0) / d.budget) * 100 
                    ELSE 0 
                END as usage_percentage
            FROM departments d
            LEFT JOIN employees e ON d.id = e.department_id
            GROUP BY d.id, d.name, d.budget
            ORDER BY usage_percentage DESC
        '''
        rows = self.db.execute_query(query)
        return [dict(row) for row in rows]
    
    def search_employees(self, search_term: str) -> List[Dict]:
        """Search employees by name or email"""
        query = '''
            SELECT e.id, e.name, e.email, e.salary, d.name as department
            FROM employees e
            JOIN departments d ON e.department_id = d.id
            WHERE e.name LIKE ? OR e.email LIKE ?
            ORDER BY e.name
        '''
        search_pattern = f'%{search_term}%'
        rows = self.db.execute_query(query, (search_pattern, search_pattern))
        return [dict(row) for row in rows]

# Usage example
db = DatabaseManager('company.db')
emp_manager = EmployeeManager(db)

# Add departments
it_dept = emp_manager.add_department('IT', 500000)
hr_dept = emp_manager.add_department('HR', 200000)
sales_dept = emp_manager.add_department('Sales', 300000)

# Add employees
emp_manager.add_employee('John Doe', 'john@company.com', it_dept, 75000)
emp_manager.add_employee('Jane Smith', 'jane@company.com', it_dept, 80000)
emp_manager.add_employee('Bob Johnson', 'bob@company.com', hr_dept, 55000)
emp_manager.add_employee('Alice Brown', 'alice@company.com', sales_dept, 65000)

# Query data
it_employees = emp_manager.get_employees_by_department('IT')
print("IT Department Employees:")
for emp in it_employees:
    print(f"- {emp['name']}: ${emp['salary']:,}")

# Get statistics
stats = emp_manager.get_salary_statistics()
print(f"\nSalary Statistics:")
print(f"Average: ${stats['avg_salary']:,.2f}")
print(f"Range: ${stats['min_salary']:,} - ${stats['max_salary']:,}")

# Budget analysis
budget_usage = emp_manager.get_department_budget_usage()
print("\nDepartment Budget Usage:")
for dept in budget_usage:
    print(f"{dept['department']}: {dept['usage_percentage']:.1f}% used")
```

### Data Migration and Backup

```python
import json
import csv
from datetime import datetime

class DataMigrator:
    """Handle data import/export and migrations"""
    
    def __init__(self, db_manager: DatabaseManager):
        self.db = db_manager
    
    def export_to_json(self, table: str, filename: str):
        """Export table data to JSON"""
        query = f"SELECT * FROM {table}"
        rows = self.db.execute_query(query)
        
        data = [dict(row) for row in rows]
        
        with open(filename, 'w') as f:
            json.dump(data, f, indent=2, default=str)
        
        print(f"Exported {len(data)} records from {table} to {filename}")
    
    def import_from_json(self, table: str, filename: str):
        """Import data from JSON file"""
        with open(filename, 'r') as f:
            data = json.load(f)
        
        if not data:
            print("No data to import")
            return
        
        # Get column names from first record
        columns = list(data[0].keys())
        placeholders = ', '.join(['?' for _ in columns])
        query = f"INSERT INTO {table} ({', '.join(columns)}) VALUES ({placeholders})"
        
        # Prepare data for bulk insert
        values = [tuple(row[col] for col in columns) for row in data]
        
        affected = self.db.execute_many(query, values)
        print(f"Imported {affected} records to {table}")
    
    def export_to_csv(self, table: str, filename: str):
        """Export table data to CSV"""
        query = f"SELECT * FROM {table}"
        rows = self.db.execute_query(query)
        
        if not rows:
            print(f"No data in table {table}")
            return
        
        with open(filename, 'w', newline='') as f:
            writer = csv.writer(f)
            # Write header
            writer.writerow(rows[0].keys())
            # Write data
            for row in rows:
                writer.writerow(row)
        
        print(f"Exported {len(rows)} records from {table} to {filename}")

# Usage example
migrator = DataMigrator(db)
migrator.export_to_json('employees', 'employees_backup.json')
migrator.export_to_csv('departments', 'departments_export.csv')

---

## Module 6: Automation Using APIs and Requests (3 Hours)

### Theory: Understanding REST APIs

REST (Representational State Transfer) APIs are web services that use HTTP methods to perform operations on resources. They follow a stateless, client-server architecture.

#### HTTP Methods

- **GET**: Retrieve data from server
- **POST**: Create new resource
- **PUT**: Update existing resource (complete replacement)
- **PATCH**: Partial update of resource
- **DELETE**: Remove resource

#### Status Codes

- **200**: OK - Request successful
- **201**: Created - Resource created successfully
- **400**: Bad Request - Invalid request
- **401**: Unauthorized - Authentication required
- **404**: Not Found - Resource doesn't exist
- **500**: Internal Server Error - Server error

### Basic Requests Usage

```python
import requests
import json
from typing import Dict, List, Optional
import time

# Simple GET request
response = requests.get('https://api.github.com/users/octocat')
if response.status_code == 200:
    user_data = response.json()
    print(f"User: {user_data['name']}")
else:
    print(f"Error: {response.status_code}")

# POST request with data
data = {
    'name': 'John Doe',
    'email': 'john@example.com'
}
response = requests.post('https://api.example.com/users', json=data)

# Headers and authentication
headers = {
    'Authorization': 'Bearer your-token-here',
    'Content-Type': 'application/json'
}
response = requests.get('https://api.example.com/protected', headers=headers)
```

### API Client Class

```python
import requests
import json
import logging
from typing import Dict, List, Optional, Any
from urllib.parse import urljoin
import time

class APIClient:
    """Generic API client with error handling and rate limiting"""
    
    def __init__(self, base_url: str, api_key: str = None, rate_limit: float = 1.0):
        self.base_url = base_url.rstrip('/')
        self.api_key = api_key
        self.rate_limit = rate_limit  # Seconds between requests
        self.last_request_time = 0
        self.session = requests.Session()
        self.logger = logging.getLogger(__name__)
        
        # Set default headers
        self.session.headers.update({
            'Content-Type': 'application/json',
            'User-Agent': 'Python-API-Client/1.0'
        })
        
        if api_key:
            self.session.headers.update({
                'Authorization': f'Bearer {api_key}'
            })
    
    def _wait_for_rate_limit(self):
        """Ensure rate limiting is respected"""
        current_time = time.time()
        time_since_last = current_time - self.last_request_time
        if time_since_last < self.rate_limit:
            time.sleep(self.rate_limit - time_since_last)
        self.last_request_time = time.time()
    
    def _make_request(self, method: str, endpoint: str, **kwargs) -> requests.Response:
        """Make HTTP request with error handling"""
        self._wait_for_rate_limit()
        
        url = urljoin(self.base_url + '/', endpoint.lstrip('/'))
        
        try:
            response = self.session.request(method, url, **kwargs)
            self.logger.info(f"{method} {url} - Status: {response.status_code}")
            response.raise_for_status()
            return response
        except requests.exceptions.HTTPError as e:
            self.logger.error(f"HTTP Error: {e}")
            raise
        except requests.exceptions.ConnectionError as e:
            self.logger.error(f"Connection Error: {e}")
            raise
        except requests.exceptions.Timeout as e:
            self.logger.error(f"Timeout Error: {e}")
            raise
        except requests.exceptions.RequestException as e:
            self.logger.error(f"Request Error: {e}")
            raise
    
    def get(self, endpoint: str, params: Dict = None) -> Dict:
        """GET request"""
        response = self._make_request('GET', endpoint, params=params)
        return response.json() if response.content else {}
    
    def post(self, endpoint: str, data: Dict = None, json_data: Dict = None) -> Dict:
        """POST request"""
        kwargs = {}
        if json_data:
            kwargs['json'] = json_data
        elif data:
            kwargs['data'] = data
        
        response = self._make_request('POST', endpoint, **kwargs)
        return response.json() if response.content else {}
    
    def put(self, endpoint: str, data: Dict = None, json_data: Dict = None) -> Dict:
        """PUT request"""
        kwargs = {}
        if json_data:
            kwargs['json'] = json_data
        elif data:
            kwargs['data'] = data
        
        response = self._make_request('PUT', endpoint, **kwargs)
        return response.json() if response.content else {}
    
    def delete(self, endpoint: str) -> bool:
        """DELETE request"""
        response = self._make_request('DELETE', endpoint)
        return response.status_code in [200, 204]
    
    def download_file(self, endpoint: str, filename: str, chunk_size: int = 8192):
        """Download file from API"""
        response = self._make_request('GET', endpoint, stream=True)
        
        with open(filename, 'wb') as f:
            for chunk in response.iter_content(chunk_size=chunk_size):
                if chunk:
                    f.write(chunk)
        
        self.logger.info(f"Downloaded {filename}")

# Usage example
client = APIClient('https://api.github.com', rate_limit=0.5)
user = client.get('/users/octocat')
print(f"User: {user['name']}, Followers: {user['followers']}")
```

### GitHub API Integration Example

```python
class GitHubClient(APIClient):
    """Specialized GitHub API client"""
    
    def __init__(self, token: str = None):
        super().__init__('https://api.github.com', token)
    
    def get_user(self, username: str) -> Dict:
        """Get user information"""
        return self.get(f'/users/{username}')
    
    def get_user_repos(self, username: str, per_page: int = 30) -> List[Dict]:
        """Get user repositories"""
        repos = []
        page = 1
        
        while True:
            params = {'per_page': per_page, 'page': page}
            response = self.get(f'/users/{username}/repos', params=params)
            
            if not response:
                break
            
            repos.extend(response)
            
            if len(response) < per_page:
                break
            
            page += 1
        
        return repos
    
    def get_repo_languages(self, owner: str, repo: str) -> Dict:
        """Get repository languages"""
        return self.get(f'/repos/{owner}/{repo}/languages')
    
    def get_repo_commits(self, owner: str, repo: str, since: str = None) -> List[Dict]:
        """Get repository commits"""
        params = {}
        if since:
            params['since'] = since
        
        return self.get(f'/repos/{owner}/{repo}/commits', params=params)
    
    def create_issue(self, owner: str, repo: str, title: str, body: str = None, labels: List[str] = None) -> Dict:
        """Create an issue"""
        data = {'title': title}
        if body:
            data['body'] = body
        if labels:
            data['labels'] = labels
        
        return self.post(f'/repos/{owner}/{repo}/issues', json_data=data)

# Usage example
github = GitHubClient('your-github-token')

# Get user info
user = github.get_user('octocat')
print(f"User: {user['name']}")
print(f"Public repos: {user['public_repos']}")

# Get repositories
repos = github.get_user_repos('octocat')
for repo in repos[:5]:  # First 5 repos
    print(f"Repository: {repo['name']} - Stars: {repo['stargazers_count']}")
    
    # Get languages for each repo
    languages = github.get_repo_languages('octocat', repo['name'])
    if languages:
        main_language = max(languages.items(), key=lambda x: x[1])[0]
        print(f"  Main language: {main_language}")
```

### Weather API Integration

```python
class WeatherClient(APIClient):
    """Weather API client example"""
    
    def __init__(self, api_key: str):
        super().__init__('https://api.openweathermap.org/data/2.5', api_key)
    
    def get_current_weather(self, city: str, units: str = 'metric') -> Dict:
        """Get current weather for a city"""
        params = {
            'q': city,
            'appid': self.api_key,
            'units': units
        }
        return self.get('/weather', params=params)
    
    def get_forecast(self, city: str, days: int = 5, units: str = 'metric') -> Dict:
        """Get weather forecast"""
        params = {
            'q': city,
            'appid': self.api_key,
            'units': units,
            'cnt': days * 8  # 8 forecasts per day (3-hour intervals)
        }
        return self.get('/forecast', params=params)
    
    def get_weather_by_coordinates(self, lat: float, lon: float, units: str = 'metric') -> Dict:
        """Get weather by latitude and longitude"""
        params = {
            'lat': lat,
            'lon': lon,
            'appid': self.api_key,
            'units': units
        }
        return self.get('/weather', params=params)

# Usage example (requires OpenWeatherMap API key)
# weather = WeatherClient('your-api-key')
# current = weather.get_current_weather('London')
# print(f"Temperature in London: {current['main']['temp']}°C")
```

### API Data Processing and Storage

```python
import sqlite3
from datetime import datetime, timedelta

class WeatherDataManager:
    """Manage weather data collection and storage"""
    
    def __init__(self, db_path: str, api_key: str):
        self.db = DatabaseManager(db_path)
        self.weather_client = WeatherClient(api_key)
        self._setup_database()
    
    def _setup_database(self):
        """Setup weather database tables"""
        self.db.create_table('cities', '''
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL UNIQUE,
            country TEXT,
            latitude REAL,
            longitude REAL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        ''')
        
        self.db.create_table('weather_data', '''
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            city_id INTEGER,
            temperature REAL,
            humidity INTEGER,
            pressure REAL,
            description TEXT,
            wind_speed REAL,
            recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (city_id) REFERENCES cities (id)
        ''')
    
    def add_city(self, name: str, country: str = None) -> int:
        """Add a city to monitor"""
        # Get city coordinates from weather API
        try:
            weather_data = self.weather_client.get_current_weather(name)
            coordinates = weather_data['coord']
            
            city_data = {
                'name': name,
                'country': country or weather_data['sys']['country'],
                'latitude': coordinates['lat'],
                'longitude': coordinates['lon']
            }
            
            return self.db.insert_record('cities', city_data)
        except Exception as e:
            print(f"Error adding city {name}: {e}")
            return None
    
    def collect_weather_data(self, city_id: int):
        """Collect weather data for a city"""
        # Get city info
        city_query = "SELECT * FROM cities WHERE id = ?"
        city = self.db.execute_query(city_query, (city_id,))[0]
        
        try:
            # Get current weather
            weather = self.weather_client.get_weather_by_coordinates(
                city['latitude'], city['longitude']
            )
            
            # Store weather data
            weather_data = {
                'city_id': city_id,
                'temperature': weather['main']['temp'],
                'humidity': weather['main']['humidity'],
                'pressure': weather['main']['pressure'],
                'description': weather['weather'][0]['description'],
                'wind_speed': weather['wind']['speed']
            }
            
            return self.db.insert_record('weather_data', weather_data)
        except Exception as e:
            print(f"Error collecting weather data for {city['name']}: {e}")
            return None
    
    def collect_all_cities(self):
        """Collect weather data for all monitored cities"""
        cities = self.db.execute_query("SELECT id, name FROM cities")
        
        for city in cities:
            print(f"Collecting weather data for {city['name']}...")
            self.collect_weather_data(city['id'])
            time.sleep(1)  # Rate limiting
    
    def get_latest_weather(self, city_name: str) -> Dict:
        """Get latest weather data for a city"""
        query = '''
            SELECT c.name, c.country, w.temperature, w.humidity, 
                   w.pressure, w.description, w.wind_speed, w.recorded_at
            FROM weather_data w
            JOIN cities c ON w.city_id = c.id
            WHERE c.name = ?
            ORDER BY w.recorded_at DESC
            LIMIT 1
        '''
        result = self.db.execute_query(query, (city_name,))
        return dict(result[0]) if result else None
    
    def get_temperature_trend(self, city_name: str, days: int = 7) -> List[Dict]:
        """Get temperature trend for the last N days"""
        since_date = (datetime.now() - timedelta(days=days)).isoformat()
        
        query = '''
            SELECT DATE(w.recorded_at) as date, 
                   AVG(w.temperature) as avg_temp,
                   MIN(w.temperature) as min_temp,
                   MAX(w.temperature) as max_temp
            FROM weather_data w
            JOIN cities c ON w.city_id = c.id
            WHERE c.name = ? AND w.recorded_at >= ?
            GROUP BY DATE(w.recorded_at)
            ORDER BY date
        '''
        
        results = self.db.execute_query(query, (city_name, since_date))
        return [dict(row) for row in results]

# Usage example
# weather_manager = WeatherDataManager('weather.db', 'your-api-key')
# 
# # Add cities to monitor
# weather_manager.add_city('London', 'UK')
# weather_manager.add_city('New York', 'US')
# weather_manager.add_city('Tokyo', 'JP')
# 
# # Collect weather data
# weather_manager.collect_all_cities()
# 
# # Get latest weather
# london_weather = weather_manager.get_latest_weather('London')
# if london_weather:
#     print(f"London: {london_weather['temperature']}°C, {london_weather['description']}")
```

### Web Scraping with Requests and BeautifulSoup

```python
import requests
from bs4 import BeautifulSoup
import csv
import time
from urllib.parse import urljoin, urlparse
import logging

class WebScraper:
    """Web scraper with error handling and rate limiting"""
    
    def __init__(self, rate_limit: float = 1.0):
        self.rate_limit = rate_limit
        self.last_request_time = 0
        self.session = requests.Session()
        self.logger = logging.getLogger(__name__)
        
        # Set headers to appear more like a real browser
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
        })
    
    def _wait_for_rate_limit(self):
        """Respect rate limiting"""
        current_time = time.time()
        time_since_last = current_time - self.last_request_time
        if time_since_last < self.rate_limit:
            time.sleep(self.rate_limit - time_since_last)
        self.last_request_time = time.time()
    
    def get_page(self, url: str) -> BeautifulSoup:
        """Get and parse a web page"""
        self._wait_for_rate_limit()
        
        try:
            response = self.session.get(url)
            response.raise_for_status()
            
            soup = BeautifulSoup(response.content, 'html.parser')
            self.logger.info(f"Successfully scraped {url}")
            return soup
        except requests.exceptions.RequestException as e:
            self.logger.error(f"Error scraping {url}: {e}")
            return None
    
    def extract_links(self, soup: BeautifulSoup, base_url: str) -> List[str]:
        """Extract all links from a page"""
        links = []
        for link in soup.find_all('a', href=True):
            url = urljoin(base_url, link['href'])
            links.append(url)
        return links
    
    def scrape_news_articles(self, base_url: str) -> List[Dict]:
        """Example: Scrape news articles (adjust selectors for specific site)"""
        soup = self.get_page(base_url)
        if not soup:
            return []
        
        articles = []
        
        # Adjust these selectors based on the actual website structure
        article_elements = soup.find_all('article') or soup.find_all('div', class_='article')
        
        for article in article_elements:
            try:
                title_elem = article.find('h1') or article.find('h2') or article.find('h3')
                title = title_elem.get_text(strip=True) if title_elem else 'No title'
                
                link_elem = article.find('a')
                link = urljoin(base_url, link_elem['href']) if link_elem and link_elem.get('href') else ''
                
                summary_elem = article.find('p')
                summary = summary_elem.get_text(strip=True) if summary_elem else 'No summary'
                
                articles.append({
                    'title': title,
                    'link': link,
                    'summary': summary[:200] + '...' if len(summary) > 200 else summary
                })
            except Exception as e:
                self.logger.warning(f"Error parsing article: {e}")
                continue
        
        return articles
    
    def save_to_csv(self, data: List[Dict], filename: str):
        """Save scraped data to CSV"""
        if not data:
            print("No data to save")
            return
        
        with open(filename, 'w', newline='', encoding='utf-8') as csvfile:
            fieldnames = data[0].keys()
            writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
            writer.writeheader()
            writer.writerows(data)
        
        print(f"Saved {len(data)} items to {filename}")

# Usage example
scraper = WebScraper(rate_limit=2.0)  # 2 seconds between requests

# Example scraping (adjust URL and selectors for real websites)
# articles = scraper.scrape_news_articles('https://example-news-site.com')
# scraper.save_to_csv(articles, 'news_articles.csv')
```

### Hands-on Exercise 6.1: API Monitoring Dashboard

```python
import json
import time
from datetime import datetime, timedelta
from typing import Dict, List
import threading

class APIMonitor:
    """Monitor multiple APIs and track their status"""
    
    def __init__(self, db_path: str):
        self.db = DatabaseManager(db_path)
        self.apis = {}
        self.monitoring = False
        self._setup_database()
    
    def _setup_database(self):
        """Setup monitoring database"""
        self.db.create_table('api_endpoints', '''
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL UNIQUE,
            url TEXT NOT NULL,
            method TEXT DEFAULT 'GET',
            headers TEXT,
            expected_status INTEGER DEFAULT 200,
            timeout INTEGER DEFAULT 30,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        ''')
        
        self.db.create_table('api_status_logs', '''
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            endpoint_id INTEGER,
            status_code INTEGER,
            response_time REAL,
            is_success BOOLEAN,
            error_message TEXT,
            checked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (endpoint_id) REFERENCES api_endpoints (id)
        ''')
    
    def add_endpoint(self, name: str, url: str, method: str = 'GET', 
                    headers: Dict = None, expected_status: int = 200, timeout: int = 30) -> int:
        """Add API endpoint to monitor"""
        data = {
            'name': name,
            'url': url,
            'method': method,
            'headers': json.dumps(headers or {}),
            'expected_status': expected_status,
            'timeout': timeout
        }
        return self.db.insert_record('api_endpoints', data)
    
    def check_endpoint(self, endpoint_id: int) -> Dict:
        """Check single endpoint status"""
        # Get endpoint details
        endpoint_query = "SELECT * FROM api_endpoints WHERE id = ?"
        endpoint = self.db.execute_query(endpoint_query, (endpoint_id,))[0]
        
        headers = json.loads(endpoint['headers']) if endpoint['headers'] else {}
        
        start_time = time.time()
        try:
            response = requests.request(
                method=endpoint['method'],
                url=endpoint['url'],
                headers=headers,
                timeout=endpoint['timeout']
            )
            
            response_time = time.time() - start_time
            is_success = response.status_code == endpoint['expected_status']
            
            # Log the result
            log_data = {
                'endpoint_id': endpoint_id,
                'status_code': response.status_code,
                'response_time': response_time,
                'is_success': is_success,
                'error_message': None if is_success else f"Expected {endpoint['expected_status']}, got {response.status_code}"
            }
            
        except Exception as e:
            response_time = time.time() - start_time
            log_data = {
                'endpoint_id': endpoint_id,
                'status_code': 0,
                'response_time': response_time,
                'is_success': False,
                'error_message': str(e)
            }
        
        self.db.insert_record('api_status_logs', log_data)
        return log_data
    
    def check_all_endpoints(self):
        """Check all registered endpoints"""
        endpoints = self.db.execute_query("SELECT id, name FROM api_endpoints")
        results = []
        
        for endpoint in endpoints:
            print(f"Checking {endpoint['name']}...")
            result = self.check_endpoint(endpoint['id'])
            result['name'] = endpoint['name']
            results.append(result)
        
        return results
    
    def start_monitoring(self, interval: int = 300):  # 5 minutes default
        """Start continuous monitoring"""
        self.monitoring = True
        
        def monitor_loop():
            while self.monitoring:
                print(f"[{datetime.now()}] Running API health checks...")
                self.check_all_endpoints()
                time.sleep(interval)
        
        monitor_thread = threading.Thread(target=monitor_loop)
        monitor_thread.daemon = True
        monitor_thread.start()
        print(f"Started monitoring with {interval} second intervals")
    
    def stop_monitoring(self):
        """Stop monitoring"""
        self.monitoring = False
        print("Stopped monitoring")
    
    def get_endpoint_stats(self, endpoint_name: str, hours: int = 24) -> Dict:
        """Get statistics for an endpoint"""
        since_time = (datetime.now() - timedelta(hours=hours)).isoformat()
        
        query = '''
            SELECT 
                COUNT(*) as total_checks,
                COUNT(CASE WHEN is_success = 1 THEN 1 END) as successful_checks,
                AVG(response_time) as avg_response_time,
                MIN(response_time) as min_response_time,
                MAX(response_time) as max_response_time
            FROM api_status_logs l
            JOIN api_endpoints e ON l.endpoint_id = e.id
            WHERE e.name = ? AND l.checked_at >= ?
        '''
        
        result = self.db.execute_query(query, (endpoint_name, since_time))[0]
        stats = dict(result)
        
        if stats['total_checks'] > 0:
            stats['uptime_percentage'] = (stats['successful_checks'] / stats['total_checks']) * 100
        else:
            stats['uptime_percentage'] = 0
        
        return stats
    
    def generate_status_report(self, filename: str = None):
        """Generate status report"""
        endpoints = self.db.execute_query("SELECT name FROM api_endpoints")
        
        report = ["API Status Report"]
        report.append("=" * 50)
        report.append(f"Generated at: {datetime.now()}")
        report.append("")
        
        for endpoint in endpoints:
            name = endpoint['name']
            stats = self.get_endpoint_stats(name)
            
            report.append(f"Endpoint: {name}")
            report.append(f"  Uptime: {stats['uptime_percentage']:.2f}%")
            report.append(f"  Total Checks: {stats['total_checks']}")
            report.append(f"  Avg Response Time: {stats['avg_response_time']:.3f}s")
            report.append("")
        
        report_text = "\n".join(report)
        
        if filename:
            with open(filename, 'w') as f:
                f.write(report_text)
            print(f"Report saved to {filename}")
        else:
            print(report_text)

# Usage example
monitor = APIMonitor('api_monitor.db')

# Add endpoints to monitor
monitor.add_endpoint('GitHub API', 'https://api.github.com')
monitor.add_endpoint('JSONPlaceholder', 'https://jsonplaceholder.typicode.com/posts/1')
monitor.add_endpoint('HTTPBin', 'https://httpbin.org/status/200')

# Run one-time check
results = monitor.check_all_endpoints()
for result in results:
    status = "✓" if result['is_success'] else "✗"
    print(f"{status} {result['name']}: {result['response_time']:.3f}s")

# Generate report
monitor.generate_status_report('api_status_report.txt')

# Start continuous monitoring (uncomment to use)
# monitor.start_monitoring(interval=60)  # Check every minute
```

---

## Module 8: Logging and Debugging (1 Hour)

### Theory: Python Logging System

Python's logging module provides a flexible framework for emitting log messages from applications. It supports different log levels and can output to multiple destinations.

#### Log Levels

- **DEBUG**: Detailed information for debugging
- **INFO**: General information about program execution
- **WARNING**: Something unexpected happened, but the program continues
- **ERROR**: A serious problem occurred
- **CRITICAL**: A very serious error occurred

### Basic Logging Setup

```python
import logging
from datetime import datetime

# Basic logging configuration
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

# Create logger
logger = logging.getLogger(__name__)

# Log messages at different levels
logger.debug("This is a debug message")
logger.info("This is an info message")
logger.warning("This is a warning message")
logger.error("This is an error message")
logger.critical("This is a critical message")
```

### Advanced Logging Configuration

```python
import logging
import logging.handlers
import os
from datetime import datetime

class CustomLogger:
    """Custom logging setup with file rotation and multiple handlers"""
    
    def __init__(self, name: str, log_dir: str = 'logs'):
        self.name = name
        self.log_dir = log_dir
        self.logger = logging.getLogger(name)
        self.logger.setLevel(logging.DEBUG)
        
        # Create log directory if it doesn't exist
        os.makedirs(log_dir, exist_ok=True)
        
        # Prevent adding handlers multiple times
        if not self.logger.handlers:
            self._setup_handlers()
    
    def _setup_handlers(self):
        """Setup different logging handlers"""
        
        # Console handler
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.INFO)
        console_format = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        console_handler.setFormatter(console_format)
        self.logger.addHandler(console_handler)
        
        # File handler with rotation
        file_handler = logging.handlers.RotatingFileHandler(
            os.path.join(self.log_dir, f'{self.name}.log'),
            maxBytes=10*1024*1024,  # 10MB
            backupCount=5
        )
        file_handler.setLevel(logging.DEBUG)
        file_format = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(funcName)s:%(lineno)d - %(message)s'
        )
        file_handler.setFormatter(file_format)
        self.logger.addHandler(file_handler)
        
        # Error handler (separate file for errors)
        error_handler = logging.handlers.RotatingFileHandler(
            os.path.join(self.log_dir, f'{self.name}_errors.log'),
            maxBytes=5*1024*1024,  # 5MB
            backupCount=3
        )
        error_handler.setLevel(logging.ERROR)
        error_handler.setFormatter(file_format)
        self.logger.addHandler(error_handler)
    
    def get_logger(self):
        """Get the configured logger"""
        return self.logger

# Usage example
custom_logger = CustomLogger('myapp')
logger = custom_logger.get_logger()

logger.info("Application started")
logger.error("An error occurred")
```

### Contextual Logging with Decorators

```python
import functools
import time
from typing import Any, Callable

def log_execution(logger):
    """Decorator to log function execution"""
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            func_name = func.__name__
            logger.info(f"Starting execution of {func_name}")
            
            start_time = time.time()
            try:
                result = func(*args, **kwargs)
                execution_time = time.time() - start_time
                logger.info(f"Completed {func_name} in {execution_time:.3f} seconds")
                return result
            except Exception as e:
                execution_time = time.time() - start_time
                logger.error(f"Error in {func_name} after {execution_time:.3f} seconds: {e}")
                raise
        return wrapper
    return decorator

def log_errors(logger):
    """Decorator to log errors with context"""
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            try:
                return func(*args, **kwargs)
            except Exception as e:
                logger.error(
                    f"Error in {func.__name__}: {e}",
                    extra={
                        'function': func.__name__,
                        'args': str(args)[:100],  # Limit length
                        'kwargs': str(kwargs)[:100]
                    }
                )
                raise
        return wrapper
    return decorator

# Usage example
logger = CustomLogger('decorated_app').get_logger()

@log_execution(logger)
@log_errors(logger)
def process_data(data_list):
    """Example function with logging decorators"""
    if not data_list:
        raise ValueError("Data list cannot be empty")
    
    processed = []
    for item in data_list:
        processed.append(item * 2)
    
    return processed

# Test the decorated function
try:
    result = process_data([1, 2, 3, 4])
    logger.info(f"Processing result: {result}")
    
    # This will cause an error and be logged
    result = process_data([])
except ValueError as e:
    logger.warning(f"Expected error handled: {e}")
```

### Application-Wide Logging System

```python
import logging
import logging.config
import json
import os
from datetime import datetime

class ApplicationLogger:
    """Centralized logging system for applications"""
    
    def __init__(self, config_file: str = None):
        self.config_file = config_file
        self.loggers = {}
        
        if config_file and os.path.exists(config_file):
            self._load_config_from_file()
        else:
            self._setup_default_config()
    
    def _setup_default_config(self):
        """Setup default logging configuration"""
        config = {
            'version': 1,
            'disable_existing_loggers': False,
            'formatters': {
                'standard': {
                    'format': '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
                },
                'detailed': {
                    'format': '%(asctime)s - %(name)s - %(levelname)s - %(module)s - %(funcName)s:%(lineno)d - %(message)s'
                }
            },
            'handlers': {
                'console': {
                    'level': 'INFO',
                    'class': 'logging.StreamHandler',
                    'formatter': 'standard',
                    'stream': 'ext://sys.stdout'
                },
                'file': {
                    'level': 'DEBUG',
                    'class': 'logging.handlers.RotatingFileHandler',
                    'formatter': 'detailed',
                    'filename': 'logs/application.log',
                    'maxBytes': 10485760,  # 10MB
                    'backupCount': 5
                },
                'error_file': {
                    'level': 'ERROR',
                    'class': 'logging.handlers.RotatingFileHandler',
                    'formatter': 'detailed',
                    'filename': 'logs/errors.log',
                    'maxBytes': 5242880,  # 5MB
                    'backupCount': 3
                }
            },
            'loggers': {
                '': {  # Root logger
                    'level': 'DEBUG',
                    'handlers': ['console', 'file', 'error_file'],
                    'propagate': False
                }
            }
        }
        
        # Create logs directory
        os.makedirs('logs', exist_ok=True)
        
        logging.config.dictConfig(config)
    
    def _load_config_from_file(self):
        """Load logging configuration from file"""
        with open(self.config_file, 'r') as f:
            config = json.load(f)
        logging.config.dictConfig(config)
    
    def get_logger(self, name: str = None) -> logging.Logger:
        """Get a logger instance"""
        if name is None:
            name = __name__
        
        if name not in self.loggers:
            self.loggers[name] = logging.getLogger(name)
        
        return self.loggers[name]
    
    def log_system_info(self):
        """Log system information"""
        logger = self.get_logger('system')
        import platform
        import sys
        
        logger.info(f"Python version: {sys.version}")
        logger.info(f"Platform: {platform.platform()}")
        logger.info(f"Architecture: {platform.architecture()}")
        logger.info(f"Processor: {platform.processor()}")

# Usage example
app_logger = ApplicationLogger()
logger = app_logger.get_logger('main')

app_logger.log_system_info()
logger.info("Application initialized")
```

### Debugging Techniques

```python
import pdb
import traceback
import inspect
from typing import Any

class Debugger:
    """Debugging utilities and helpers"""
    
    @staticmethod
    def debug_on_error(func):
        """Decorator to start debugger on exception"""
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                print(f"Exception in {func.__name__}: {e}")
                print("Starting debugger...")
                pdb.post_mortem()
                raise
        return wrapper
    
    @staticmethod
    def trace_calls(func):
        """Decorator to trace function calls"""
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print(f"TRACE: Calling {func.__name__}")
            print(f"TRACE: Args: {args}")
            print(f"TRACE: Kwargs: {kwargs}")
            
            result = func(*args, **kwargs)
            
            print(f"TRACE: {func.__name__} returned: {result}")
            return result
        return wrapper
    
    @staticmethod
    def print_stack_trace():
        """Print current stack trace"""
        print("Current stack trace:")
        for line in traceback.format_stack():
            print(line.strip())
    
    @staticmethod
    def inspect_object(obj: Any, detailed: bool = False):
        """Inspect an object and print its attributes"""
        print(f"Inspecting object of type: {type(obj).__name__}")
        
        if detailed:
            print("\nAll attributes and methods:")
            for attr_name in dir(obj):
                attr_value = getattr(obj, attr_name)
                print(f"  {attr_name}: {type(attr_value).__name__}")
        else:
            print("\nPublic attributes and methods:")
            for attr_name in dir(obj):
                if not attr_name.startswith('_'):
                    attr_value = getattr(obj, attr_name)
                    print(f"  {attr_name}: {type(attr_value).__name__}")
    
    @staticmethod
    def get_caller_info():
        """Get information about the calling function"""
        frame = inspect.currentframe().f_back
        return {
            'filename': frame.f_code.co_filename,
            'function': frame.f_code.co_name,
            'line_number': frame.f_lineno,
            'local_vars': dict(frame.f_locals)
        }

# Usage examples
@Debugger.trace_calls
def sample_function(x, y):
    """Sample function for demonstration"""
    return x + y

# Test tracing
result = sample_function(5, 3)

# Inspect an object
sample_dict = {'key1': 'value1', 'key2': 42}
Debugger.inspect_object(sample_dict)

# Get caller information
def calling_function():
    caller_info = Debugger.get_caller_info()
    print(f"Called from: {caller_info['function']} at line {caller_info['line_number']}")

calling_function()
```

### Performance Logging and Monitoring

```python
import time
import psutil
import threading
from collections import defaultdict
import statistics

class PerformanceMonitor:
    """Monitor application performance and log metrics"""
    
    def __init__(self, logger_name: str = 'performance'):
        self.logger = logging.getLogger(logger_name)
        self.metrics = defaultdict(list)
        self.monitoring = False
        self.monitor_thread = None
    
    def time_function(self, func_name: str = None):
        """Decorator to time function execution"""
        def decorator(func):
            name = func_name or func.__name__
            
            @functools.wraps(func)
            def wrapper(*args, **kwargs):
                start_time = time.time()
                result = func(*args, **kwargs)
                execution_time = time.time() - start_time
                
                self.metrics[f'{name}_execution_time'].append(execution_time)
                self.logger.info(f"{name} executed in {execution_time:.4f} seconds")
                
                return result
            return wrapper
        return decorator
    
    def monitor_memory(self):
        """Monitor memory usage"""
        process = psutil.Process()
        memory_info = process.memory_info()
        
        memory_mb = memory_info.rss / 1024 / 1024
        self.metrics['memory_usage_mb'].append(memory_mb)
        
        return memory_mb
    
    def monitor_cpu(self):
        """Monitor CPU usage"""
        cpu_percent = psutil.cpu_percent(interval=1)
        self.metrics['cpu_usage_percent'].append(cpu_percent)
        
        return cpu_percent
    
    def start_system_monitoring(self, interval: int = 30):
        """Start continuous system monitoring"""
        def monitor_loop():
            while self.monitoring:
                memory_usage = self.monitor_memory()
                cpu_usage = self.monitor_cpu()
                
                self.logger.info(f"System metrics - Memory: {memory_usage:.1f}MB, CPU: {cpu_usage:.1f}%")
                
                time.sleep(interval)
        
        self.monitoring = True
        self.monitor_thread = threading.Thread(target=monitor_loop)
        self.monitor_thread.daemon = True
        self.monitor_thread.start()
        
        self.logger.info(f"Started system monitoring with {interval}s interval")
    
    def stop_system_monitoring(self):
        """Stop system monitoring"""
        self.monitoring = False
        if self.monitor_thread:
            self.monitor_thread.join()
        self.logger.info("Stopped system monitoring")
    
    def get_performance_summary(self) -> dict:
        """Get performance metrics summary"""
        summary = {}
        
        for metric_name, values in self.metrics.items():
            if values:
                summary[metric_name] = {
                    'count': len(values),
                    'average': statistics.mean(values),
                    'min': min(values),
                    'max': max(values),
                    'median': statistics.median(values)
                }
        
        return summary
    
    def log_performance_summary(self):
        """Log performance summary"""
        summary = self.get_performance_summary()
        
        self.logger.info("Performance Summary")
        self.logger.info("=" * 50)
        
        for metric, stats in summary.items():
            self.logger.info(f"{metric}:")
            self.logger.info(f"  Count: {stats['count']}")
            self.logger.info(f"  Average: {stats['average']:.4f}")
            self.logger.info(f"  Min: {stats['min']:.4f}")
            self.logger.info(f"  Max: {stats['max']:.4f}")
            self.logger.info(f"  Median: {stats['median']:.4f}")

# Usage example
perf_monitor = PerformanceMonitor()

@perf_monitor.time_function()
def cpu_intensive_task():
    """Simulate CPU intensive work"""
    total = 0
    for i in range(1000000):
        total += i ** 2
    return total

@perf_monitor.time_function('database_query')
def simulate_database_query():
    """Simulate database query"""
    time.sleep(0.1)  # Simulate query time
    return "Query result"

# Start monitoring
perf_monitor.start_system_monitoring(interval=5)

# Run some tasks
for i in range(3):
    result = cpu_intensive_task()
    db_result = simulate_database_query()
    time.sleep(1)

# Stop monitoring and show summary
perf_monitor.stop_system_monitoring()
perf_monitor.log_performance_summary()
```

### Hands-on Exercise 8.1: Complete Logging System

```python
import logging
import logging.handlers
import json
import os
import time
from datetime import datetime
from typing import Dict, Any
import smtplib
from email.mime.text import MIMEText

class ProductionLoggingSystem:
    """Production-ready logging system with alerting"""
    
    def __init__(self, app_name: str, config: Dict[str, Any] = None):
        self.app_name = app_name
        self.config = config or self._default_config()
        self.loggers = {}
        self.alert_handlers = []
        
        self._setup_logging()
        self._setup_alerts()
    
    def _default_config(self) -> Dict[str, Any]:
        """Default logging configuration"""
        return {
            'log_level': 'INFO',
            'log_dir': 'logs',
            'max_file_size': 10 * 1024 * 1024,  # 10MB
            'backup_count': 5,
            'enable_email_alerts': False,
            'email_config': {
                'smtp_server': 'localhost',
                'smtp_port': 587,
                'username': '',
                'password': '',
                'from_email': '',
                'to_emails': []
            },
            'alert_threshold': 'ERROR'
        }
    
    def _setup_logging(self):
        """Setup logging configuration"""
        log_dir = self.config['log_dir']
        os.makedirs(log_dir, exist_ok=True)
        
        # Create formatters
        detailed_formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(module)s:%(funcName)s:%(lineno)d - %(message)s'
        )
        
        simple_formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s'
        )
        
        # Setup root logger
        root_logger = logging.getLogger()
        root_logger.setLevel(getattr(logging, self.config['log_level']))
        
        # Console handler
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.INFO)
        console_handler.setFormatter(simple_formatter)
        root_logger.addHandler(console_handler)
        
        # Main application file handler
        app_handler = logging.handlers.RotatingFileHandler(
            os.path.join(log_dir, f'{self.app_name}.log'),
            maxBytes=self.config['max_file_size'],
            backupCount=self.config['backup_count']
        )
        app_handler.setLevel(logging.DEBUG)
        app_handler.setFormatter(detailed_formatter)
        root_logger.addHandler(app_handler)
        
        # Error file handler
        error_handler = logging.handlers.RotatingFileHandler(
            os.path.join(log_dir, f'{self.app_name}_errors.log'),
            maxBytes=self.config['max_file_size'] // 2,
            backupCount=self.config['backup_count']
        )
        error_handler.setLevel(logging.ERROR)
        error_handler.setFormatter(detailed_formatter)
        root_logger.addHandler(error_handler)
        
        # Performance log handler
        perf_handler = logging.handlers.RotatingFileHandler(
            os.path.join(log_dir, f'{self.app_name}_performance.log'),
            maxBytes=self.config['max_file_size'],
            backupCount=self.config['backup_count']
        )
        perf_handler.setLevel(logging.INFO)
        perf_formatter = logging.Formatter(
            '%(asctime)s - PERF - %(message)s'
        )
        perf_handler.setFormatter(perf_formatter)
        
        # Create performance logger
        perf_logger = logging.getLogger(f'{self.app_name}.performance')
        perf_logger.addHandler(perf_handler)
        perf_logger.setLevel(logging.INFO)
        perf_logger.propagate = False
    
    def _setup_alerts(self):
        """Setup alert mechanisms"""
        if self.config['enable_email_alerts']:
            email_handler = self._create_email_handler()
            if email_handler:
                self.alert_handlers.append(email_handler)
                
                # Add email handler to root logger for critical errors
                root_logger = logging.getLogger()
                root_logger.addHandler(email_handler)
    
    def _create_email_handler(self):
        """Create email alert handler"""
        try:
            from logging.handlers import SMTPHandler
            
            email_config = self.config['email_config']
            
            mail_handler = SMTPHandler(
                mailhost=(email_config['smtp_server'], email_config['smtp_port']),
                fromaddr=email_config['from_email'],
                toaddrs=email_config['to_emails'],
                subject=f'ALERT: {self.app_name} Error',
                credentials=(email_config['username'], email_config['password']),
                secure=()
            )
            
            mail_handler.setLevel(getattr(logging, self.config['alert_threshold']))
            
            return mail_handler
        except Exception as e:
            print(f"Failed to setup email alerts: {e}")
            return None
    
    def get_logger(self, name: str = None) -> logging.Logger:
        """Get application logger"""
        if name is None:
            name = self.app_name
        
        if name not in self.loggers:
            self.loggers[name] = logging.getLogger(name)
        
        return self.loggers[name]
    
    def get_performance_logger(self) -> logging.Logger:
        """Get performance logger"""
        return logging.getLogger(f'{self.app_name}.performance')
    
    def log_application_start(self):
        """Log application startup information"""
        logger = self.get_logger()
        
        logger.info(f"{self.app_name} starting up")
        logger.info(f"Log level: {self.config['log_level']}")
        logger.info(f"Log directory: {self.config['log_dir']}")
        
        # Log system information
        import platform
        import sys
        
        logger.info(f"Python version: {sys.version}")
        logger.info(f"Platform: {platform.platform()}")
        logger.info(f"Working directory: {os.getcwd()}")
    
    def log_application_shutdown(self):
        """Log application shutdown"""
        logger = self.get_logger()
        logger.info(f"{self.app_name} shutting down")
    
    def create_audit_log(self, user: str, action: str, resource: str, details: Dict = None):
        """Create audit log entry"""
        audit_logger = logging.getLogger(f'{self.app_name}.audit')
        
        audit_entry = {
            'timestamp': datetime.now().isoformat(),
            'user': user,
            'action': action,
            'resource': resource,
            'details': details or {}
        }
        
        audit_logger.info(json.dumps(audit_entry))

# Usage example
logging_config = {
    'log_level': 'DEBUG',
    'log_dir': 'app_logs',
    'enable_email_alerts': False,  # Set to True for email alerts
    'alert_threshold': 'CRITICAL'
}

logging_system = ProductionLoggingSystem('MyApp', logging_config)
logger = logging_system.get_logger()
perf_logger = logging_system.get_performance_logger()

# Log application startup
logging_system.log_application_start()

# Example usage
logger.info("Processing user request")
logger.warning("Low disk space detected")

# Performance logging
start_time = time.time()
time.sleep(0.1)  # Simulate work
end_time = time.time()
perf_logger.info(f"Operation completed in {end_time - start_time:.3f} seconds")

# Audit logging
logging_system.create_audit_log(
    user='john_doe',
    action='file_upload',
    resource='/documents/report.pdf',
    details={'file_size': 1024000, 'ip_address': '192.168.1.100'}
)

# Error with traceback
try:
    raise ValueError("Sample error for demonstration")
except ValueError:
    logger.exception("An error occurred while processing request")

# Log application shutdown
logging_system.log_application_shutdown()

print("Check the 'app_logs' directory for log files")
```

---

# Day 3: Testing & Document Processing

## Module 7: Test Automation Frameworks (Pytest) (6 Hours)

### Theory: Software Testing Fundamentals

Testing is crucial for ensuring code quality and reliability. Python's pytest framework provides powerful features for writing and running tests.

#### Types of Testing

1. **Unit Tests**: Test individual functions/methods
2. **Integration Tests**: Test component interactions
3. **Functional Tests**: Test complete features
4. **Performance Tests**: Test speed and resource usage

#### Testing Best Practices

- Write tests before or alongside code (TDD)
- Test edge cases and error conditions
- Keep tests simple and focused
- Use descriptive test names
- Ensure tests are independent and repeatable

### Getting Started with Pytest

```python
# test_basic.py
import pytest

def add(a, b):
    """Simple addition function"""
    return a + b

def divide(a, b):
    """Division function"""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# Basic test functions
def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0

def test_divide():
    assert divide(10, 2) == 5
    assert divide(7, 2) == 3.5

def test_divide_by_zero():
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)

# Run tests with: pytest test_basic.py -v
```

### Advanced Pytest Features

#### Fixtures

```python
# conftest.py
import pytest
import tempfile
import os
import sqlite3

@pytest.fixture
def sample_data():
    """Provide sample data for tests"""
    return [
        {'name': 'John', 'age': 30, 'city': 'New York'},
        {'name': 'Jane', 'age': 25, 'city': 'Los Angeles'},
        {'name': 'Bob', 'age': 35, 'city': 'Chicago'}
    ]

@pytest.fixture
def temp_file():
    """Create temporary file for testing"""
    fd, path = tempfile.mkstemp()
    yield path
    os.close(fd)
    os.unlink(path)

@pytest.fixture
def test_database():
    """Create temporary database for testing"""
    fd, db_path = tempfile.mkstemp(suffix='.db')
    os.close(fd)
    
    # Setup database
    conn = sqlite3.connect(db_path)
    conn.execute('''
        CREATE TABLE users (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT UNIQUE NOT NULL
        )
    ''')
    conn.commit()
    conn.close()
    
    yield db_path
    
    # Cleanup
    os.unlink(db_path)

@pytest.fixture(scope="session")
def api_client():
    """Session-scoped fixture for API client"""
    from your_api_module import APIClient
    client = APIClient("http://test-api.example.com")
    yield client
    client.close()  # Cleanup if needed
```

#### Parameterized Tests

```python
# test_parametrized.py
import pytest

@pytest.mark.parametrize("input_value,expected", [
    (1, 2),
    (2, 4),
    (3, 6),
    (4, 8),
    (0, 0),
    (-1, -2)
])
def test_double(input_value, expected):
    """Test doubling function with multiple inputs"""
    def double(x):
        return x * 2
    
    assert double(input_value) == expected

@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (10, -5, 5),
    (0, 0, 0),
    (-1, -1, -2),
    (1.5, 2.5, 4.0)
])
def test_addition(a, b, expected):
    """Test addition with various inputs"""
    assert add(a, b) == expected

# Testing with multiple parameters
@pytest.mark.parametrize("username", ["john", "jane", "admin"])
@pytest.mark.parametrize("password", ["pass123", "secret", "admin123"])
def test_login_combinations(username, password):
    """Test login with different username/password combinations"""
    result = validate_login(username, password)
    assert isinstance(result, bool)
```

### Testing Classes and Methods

```python
# user_manager.py
class UserManager:
    def __init__(self):
        self.users = {}
        self.next_id = 1
    
    def add_user(self, name, email):
        if email in [user['email'] for user in self.users.values()]:
            raise ValueError("Email already exists")
        
        user_id = self.next_id
        self.users[user_id] = {
            'id': user_id,
            'name': name,
            'email': email
        }
        self.next_id += 1
        return user_id
    
    def get_user(self, user_id):
        return self.users.get(user_id)
    
    def update_user(self, user_id, name=None, email=None):
        if user_id not in self.users:
            raise ValueError("User not found")
        
        user = self.users[user_id]
        if name:
            user['name'] = name
        if email:
            if email in [u['email'] for uid, u in self.users.items() if uid != user_id]:
                raise ValueError("Email already exists")
            user['email'] = email
        
        return user
    
    def delete_user(self, user_id):
        if user_id not in self.users:
            raise ValueError("User not found")
        del self.users[user_id]
    
    def list_users(self):
        return list(self.users.values())

# test_user_manager.py
import pytest
from user_manager import UserManager

class TestUserManager:
    """Test class for UserManager"""
    
    @pytest.fixture
    def user_manager(self):
        """Create fresh UserManager for each test"""
        return UserManager()
    
    def test_add_user(self, user_manager):
        """Test adding a new user"""
        user_id = user_manager.add_user("John Doe", "john@example.com")
        
        assert user_id == 1
        assert len(user_manager.users) == 1
        
        user = user_manager.get_user(user_id)
        assert user['name'] == "John Doe"
        assert user['email'] == "john@example.com"
    
    def test_add_duplicate_email(self, user_manager):
        """Test adding user with duplicate email raises error"""
        user_manager.add_user("John Doe", "john@example.com")
        
        with pytest.raises(ValueError, match="Email already exists"):
            user_manager.add_user("Jane Doe", "john@example.com")
    
    def test_get_nonexistent_user(self, user_manager):
        """Test getting non-existent user returns None"""
        user = user_manager.get_user(999)
        assert user is None
    
    def test_update_user(self, user_manager):
        """Test updating user information"""
        user_id = user_manager.add_user("John Doe", "john@example.com")
        
        updated_user = user_manager.update_user(user_id, name="John Smith")
        assert updated_user['name'] == "John Smith"
        assert updated_user['email'] == "john@example.com"
        
        updated_user = user_manager.update_user(user_id, email="john.smith@example.com")
        assert updated_user['email'] == "john.smith@example.com"
    
    def test_update_nonexistent_user(self, user_manager):
        """Test updating non-existent user raises error"""
        with pytest.raises(ValueError, match="User not found"):
            user_manager.update_user(999, name="New Name")
    
    def test_delete_user(self, user_manager):
        """Test deleting a user"""
        user_id = user_manager.add_user("John Doe", "john@example.com")
        assert len(user_manager.users) == 1
        
        user_manager.delete_user(user_id)
        assert len(user_manager.users) == 0
        assert user_manager.get_user(user_id) is None
    
    def test_delete_nonexistent_user(self, user_manager):
        """Test deleting non-existent user raises error"""
        with pytest.raises(ValueError, match="User not found"):
            user_manager.delete_user(999)
    
    def test_list_users(self, user_manager):
        """Test listing all users"""
        assert user_manager.list_users() == []
        
        user_manager.add_user("John Doe", "john@example.com")
        user_manager.add_user("Jane Smith", "jane@example.com")
        
        users = user_manager.list_users()
        assert len(users) == 2
        assert any(user['name'] == "John Doe" for user in users)
        assert any(user['name'] == "Jane Smith" for user in users)
```

### Mocking and Patching

```python
# external_service.py
import requests
import json

class WeatherService:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.openweathermap.org/data/2.5"
    
    def get_weather(self, city):
        """Get weather data for a city"""
        url = f"{self.base_url}/weather"
        params = {
            'q': city,
            'appid': self.api_key,
            'units': 'metric'
        }
        
        response = requests.get(url, params=params)
        response.raise_for_status()
        
        return response.json()
    
    def is_sunny(self, city):
        """Check if it's sunny in a city"""
        weather_data = self.get_weather(city)
        weather_condition = weather_data['weather'][0]['main'].lower()
        return weather_condition in ['clear', 'sunny']

# test_external_service.py
import pytest
from unittest.mock import Mock, patch, MagicMock
import requests
from external_service import WeatherService

class TestWeatherService:
    
    @pytest.fixture
    def weather_service(self):
        return WeatherService("test_api_key")
    
    @patch('external_service.requests.get')
    def test_get_weather_success(self, mock_get, weather_service):
        """Test successful weather data retrieval"""
        # Setup mock response
        mock_response = Mock()
        mock_response.json.return_value = {
            'weather': [{'main': 'Clear', 'description': 'clear sky'}],
            'main': {'temp': 25.5, 'humidity': 60},
            'name': 'London'
        }
        mock_response.raise_for_status.return_value = None
        mock_get.return_value = mock_response
        
        # Test the method
        result = weather_service.get_weather("London")
        
        # Assertions
        assert result['name'] == 'London'
        assert result['main']['temp'] == 25.5
        mock_get.assert_called_once()
        
        # Check the actual call parameters
        call_args = mock_get.call_args
        assert call_args[1]['params']['q'] == 'London'
        assert call_args[1]['params']['appid'] == 'test_api_key'
    
    @patch('external_service.requests.get')
    def test_get_weather_api_error(self, mock_get, weather_service):
        """Test API error handling"""
        mock_response = Mock()
        mock_response.raise_for_status.side_effect = requests.HTTPError("404 Not Found")
        mock_get.return_value = mock_response
        
        with pytest.raises(requests.HTTPError):
            weather_service.get_weather("InvalidCity")
    
    @patch.object(WeatherService, 'get_weather')
    def test_is_sunny_true(self, mock_get_weather, weather_service):
        """Test is_sunny returns True for clear weather"""
        mock_get_weather.return_value = {
            'weather': [{'main': 'Clear'}]
        }
        
        result = weather_service.is_sunny("London")
        assert result is True
        mock_get_weather.assert_called_once_with("London")
    
    @patch.object(WeatherService, 'get_weather')
    def test_is_sunny_false(self, mock_get_weather, weather_service):
        """Test is_sunny returns False for non-clear weather"""
        mock_get_weather.return_value = {
            'weather': [{'main': 'Rain'}]
        }
        
        result = weather_service.is_sunny("London")
        assert result is False
    
    @pytest.mark.parametrize("weather_condition,expected", [
        ("Clear", True),
        ("Sunny", True),
        ("Rain", False),
        ("Clouds", False),
        ("Snow", False)
    ])
    @patch.object(WeatherService, 'get_weather')
    def test_is_sunny_various_conditions(self, mock_get_weather, weather_service, weather_condition, expected):
        """Test is_sunny with various weather conditions"""
        mock_get_weather.return_value = {
            'weather': [{'main': weather_condition}]
        }
        
        result = weather_service.is_sunny("TestCity")
        assert result == expected
```

### Testing with Database

```python
# database_service.py
import sqlite3
from typing import List, Dict, Optional

class UserDatabase:
    def __init__(self, db_path: str):
        self.db_path = db_path
        self._init_database()
    
    def _init_database(self):
        """Initialize database tables"""
        with sqlite3.connect(self.db_path) as conn:
            conn.execute('''
                CREATE TABLE IF NOT EXISTS users (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    name TEXT NOT NULL,
                    email TEXT UNIQUE NOT NULL,
                    age INTEGER,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            ''')
            conn.commit()
    
    def add_user(self, name: str, email: str, age: int = None) -> int:
        """Add a new user"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.cursor()
            cursor.execute(
                "INSERT INTO users (name, email, age) VALUES (?, ?, ?)",
                (name, email, age)
            )
            conn.commit()
            return cursor.lastrowid
    
    def get_user(self, user_id: int) -> Optional[Dict]:
        """Get user by ID"""
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.cursor()
            cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
            row = cursor.fetchone()
            return dict(row) if row else None
    
    def get_users_by_age_range(self, min_age: int, max_age: int) -> List[Dict]:
        """Get users within age range"""
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.cursor()
            cursor.execute(
                "SELECT * FROM users WHERE age BETWEEN ? AND ? ORDER BY age",
                (min_age, max_age)
            )
            return [dict(row) for row in cursor.fetchall()]
    
    def update_user(self, user_id: int, **kwargs) -> bool:
        """Update user information"""
        if not kwargs:
            return False
        
        set_clause = ", ".join([f"{key} = ?" for key in kwargs.keys()])
        values = list(kwargs.values()) + [user_id]
        
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.cursor()
            cursor.execute(
                f"UPDATE users SET {set_clause} WHERE id = ?",
                values
            )
            conn.commit()
            return cursor.rowcount > 0
    
    def delete_user(self, user_id: int) -> bool:
        """Delete user"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.cursor()
            cursor.execute("DELETE FROM users WHERE id = ?", (user_id,))
            conn.commit()
            return cursor.rowcount > 0

# test_database_service.py
import pytest
import tempfile
import os
from database_service import UserDatabase

class TestUserDatabase:
    
    @pytest.fixture
    def db(self):
        """Create temporary database for testing"""
        fd, db_path = tempfile.mkstemp(suffix='.db')
        os.close(fd)
        
        database = UserDatabase(db_path)
        yield database
        
        # Cleanup
        os.unlink(db_path)
    
    def test_add_user(self, db):
        """Test adding a new user"""
        user_id = db.add_user("John Doe", "john@example.com", 30)
        
        assert user_id is not None
        assert user_id > 0
        
        # Verify user was added
        user = db.get_user(user_id)
        assert user is not None
        assert user['name'] == "John Doe"
        assert user['email'] == "john@example.com"
        assert user['age'] == 30
    
    def test_add_user_without_age(self, db):
        """Test adding user without age"""
        user_id = db.add_user("Jane Doe", "jane@example.com")
        
        user = db.get_user(user_id)
        assert user['age'] is None
    
    def test_get_nonexistent_user(self, db):
        """Test getting non-existent user"""
        user = db.get_user(999)
        assert user is None
    
    def test_update_user(self, db):
        """Test updating user information"""
        user_id = db.add_user("John Doe", "john@example.com", 30)
        
        # Update name and age
        success = db.update_user(user_id, name="John Smith", age=31)
        assert success is True
        
        # Verify update
        user = db.get_user(user_id)
        assert user['name'] == "John Smith"
        assert user['age'] == 31
        assert user['email'] == "john@example.com"  # Unchanged
    
    def test_update_nonexistent_user(self, db):
        """Test updating non-existent user"""
        success = db.update_user(999, name="New Name")
        assert success is False
    
    def test_delete_user(self, db):
        """Test deleting user"""
        user_id = db.add_user("John Doe", "john@example.com", 30)
        
        success = db.delete_user(user_id)
        assert success is True
        
        # Verify deletion
        user = db.get_user(user_id)
        assert user is None
    
    def test_delete_nonexistent_user(self, db):
        """Test deleting non-existent user"""
        success = db.delete_user(999)
        assert success is False
    
    def test_get_users_by_age_range(self, db):
        """Test getting users by age range"""
        # Add test users
        db.add_user("Alice", "alice@example.com", 25)
        db.add_user("Bob", "bob@example.com", 30)
        db.add_user("Charlie", "charlie@example.com", 35)
        db.add_user("Diana", "diana@example.com", 40)
        
        # Test age range query
        users = db.get_users_by_age_range(28, 37)
        assert len(users) == 2
        
        names = [user['name'] for user in users]
        assert "Bob" in names
        assert "Charlie" in names
        assert "Alice" not in names
        assert "Diana" not in names
    
    def test_database_persistence(self, db):
        """Test that data persists across operations"""
        # Add multiple users
        user_ids = []
        for i in range(5):
            user_id = db.add_user(f"User{i}", f"user{i}@example.com", 20 + i)
            user_ids.append(user_id)
        
        # Verify all users exist
        for user_id in user_ids:
            user = db.get_user(user_id)
            assert user is not None
        
        # Delete some users
        db.delete_user(user_ids[1])
        db.delete_user(user_ids[3])
        
        # Verify correct users were deleted
        assert db.get_user(user_ids[0]) is not None
        assert db.get_user(user_ids[1]) is None
        assert db.get_user(user_ids[2]) is not None
        assert db.get_user(user_ids[3]) is None
        assert db.get_user(user_ids[4]) is not None
```

### Test Coverage and Reporting

```python
# conftest.py for coverage configuration
import pytest
import coverage

@pytest.fixture(scope="session", autouse=True)
def coverage_report():
    """Generate coverage report after all tests"""
    cov = coverage.Coverage()
    cov.start()
    
    yield
    
    cov.stop()
    cov.save()
    
    print("\nCoverage Report:")
    cov.report(show_missing=True)
    
    # Generate HTML report
    cov.html_report(directory='htmlcov')
    print("HTML coverage report generated in 'htmlcov' directory")

# Custom markers in pytest.ini
"""
[tool:pytest]
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    integration: marks tests as integration tests
    unit: marks tests as unit tests
    api: marks tests that require API access
"""

# test_with_markers.py
import pytest
import time

@pytest.mark.unit
def test_fast_function():
    """Fast unit test"""
    assert 1 + 1 == 2

@pytest.mark.slow
def test_slow_function():
    """Slow test that we might want to skip"""
    time.sleep(2)
    assert True

@pytest.mark.integration
def test_database_integration(test_database):
    """Integration test with database"""
    # Test database operations
    pass

@pytest.mark.api
def test_external_api():
    """Test that requires external API"""
    # Skip if no internet connection
    pytest.importorskip("requests")
    # Test API calls
    pass

# Skip tests conditionally
@pytest.mark.skipif(os.environ.get("CI") == "true", reason="Skip in CI environment")
def test_local_only():
    """Test that only runs locally"""
    pass

# Expected to fail
@pytest.mark.xfail(reason="Known bug, fix in progress")
def test_known_bug():
    """Test for known bug"""
    assert False  # This will be marked as expected failure
```

### Testing Async Code

```python
# async_service.py
import asyncio
import aiohttp
from typing import Dict, List

class AsyncWeatherService:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://api.openweathermap.org/data/2.5"
    
    async def get_weather(self, city: str) -> Dict:
        """Get weather data asynchronously"""
        async with aiohttp.ClientSession() as session:
            url = f"{self.base_url}/weather"
            params = {
                'q': city,
                'appid': self.api_key,
                'units': 'metric'
            }
            
            async with session.get(url, params=params) as response:
                response.raise_for_status()
                return await response.json()
    
    async def get_multiple_cities_weather(self, cities: List[str]) -> Dict[str, Dict]:
        """Get weather for multiple cities concurrently"""
        tasks = [self.get_weather(city) for city in cities]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        weather_data = {}
        for city, result in zip(cities, results):
            if isinstance(result, Exception):
                weather_data[city] = {'error': str(result)}
            else:
                weather_data[city] = result
        
        return weather_data

# test_async_service.py
import pytest
import asyncio
from unittest.mock import AsyncMock, patch
import aiohttp
from async_service import AsyncWeatherService

class TestAsyncWeatherService:
    
    @pytest.fixture
    def weather_service(self):
        return AsyncWeatherService("test_api_key")
    
    @pytest.mark.asyncio
    async def test_get_weather_success(self, weather_service):
        """Test successful async weather retrieval"""
        with patch('aiohttp.ClientSession.get') as mock_get:
            # Setup async mock
            mock_response = AsyncMock()
            mock_response.json = AsyncMock(return_value={
                'weather': [{'main': 'Clear'}],
                'main': {'temp': 25.0},
                'name': 'London'
            })
            mock_response.raise_for_status = AsyncMock()
            
            # Setup context manager mock
            mock_get.return_value.__aenter__.return_value = mock_response
            
            result = await weather_service.get_weather("London")
            
            assert result['name'] == 'London'
            assert result['main']['temp'] == 25.0
    
    @pytest.mark.asyncio
    async def test_get_multiple_cities_weather(self, weather_service):
        """Test getting weather for multiple cities"""
        with patch.object(weather_service, 'get_weather') as mock_get_weather:
            mock_get_weather.side_effect = [
                {'name': 'London', 'main': {'temp': 20}},
                {'name': 'Paris', 'main': {'temp': 18}},
                {'name': 'Berlin', 'main': {'temp': 15}}
            ]
            
            cities = ['London', 'Paris', 'Berlin']
            results = await weather_service.get_multiple_cities_weather(cities)
            
            assert len(results) == 3
            assert 'London' in results
            assert 'Paris' in results
            assert 'Berlin' in results
            
            # Verify all cities were requested
            assert mock_get_weather.call_count == 3
    
    @pytest.mark.asyncio
    async def test_get_multiple_cities_with_errors(self, weather_service):
        """Test handling errors when getting multiple cities weather"""
        with patch.object(weather_service, 'get_weather') as mock_get_weather:
            mock_get_weather.side_effect = [
                {'name': 'London', 'main': {'temp': 20}},
                aiohttp.ClientError("Network error"),
                {'name': 'Berlin', 'main': {'temp': 15}}
            ]
            
            cities = ['London', 'Paris', 'Berlin']
            results = await weather_service.get_multiple_cities_weather(cities)
            
            assert 'London' in results
            assert 'error' in results['Paris']
            assert 'Berlin' in results
            assert results['Paris']['error'] == "Network error"
```

### Hands-on Exercise 7.1: Complete Test Suite

```python
# calculator.py - Simple calculator class to test
class Calculator:
    """A simple calculator with memory functionality"""
    
    def __init__(self):
        self.memory = 0
        self.history = []
    
    def add(self, a, b):
        """Add two numbers"""
        result = a + b
        self.history.append(f"{a} + {b} = {result}")
        return result
    
    def subtract(self, a, b):
        """Subtract b from a"""
        result = a - b
        self.history.append(f"{a} - {b} = {result}")
        return result
    
    def multiply(self, a, b):
        """Multiply two numbers"""
        result = a * b
        self.history.append(f"{a} * {b} = {result}")
        return result
    
    def divide(self, a, b):
        """Divide a by b"""
        if b == 0:
            raise ValueError("Cannot divide by zero")
        result = a / b
        self.history.append(f"{a} / {b} = {result}")
        return result
    
    def power(self, base, exponent):
        """Calculate base raised to exponent"""
        if exponent < 0:
            raise ValueError("Negative exponents not supported")
        result = base ** exponent
        self.history.append(f"{base} ^ {exponent} = {result}")
        return result
    
    def store_memory(self, value):
        """Store value in memory"""
        self.memory = value
    
    def recall_memory(self):
        """Recall value from memory"""
        return self.memory
    
    def clear_memory(self):
        """Clear memory"""
        self.memory = 0
    
    def clear_history(self):
        """Clear calculation history"""
        self.history = []
    
    def get_history(self):
        """Get calculation history"""
        return self.history.copy()

# test_calculator_complete.py
import pytest
import math
from calculator import Calculator

class TestCalculator:
    """Comprehensive test suite for Calculator class"""
    
    @pytest.fixture
    def calc(self):
        """Provide a fresh calculator instance for each test"""
        return Calculator()
    
    # Basic operation tests
    class TestBasicOperations:
        """Test basic arithmetic operations"""
        
        @pytest.mark.parametrize("a,b,expected", [
            (2, 3, 5),
            (-1, 1, 0),
            (0, 0, 0),
            (1.5, 2.5, 4.0),
            (-5, -3, -8)
        ])
        def test_add(self, calc, a, b, expected):
            """Test addition with various inputs"""
            result = calc.add(a, b)
            assert result == expected
            assert len(calc.history) == 1
        
        @pytest.mark.parametrize("a,b,expected", [
            (5, 3, 2),
            (0, 5, -5),
            (10, 10, 0),
            (1.5, 0.5, 1.0)
        ])
        def test_subtract(self, calc, a, b, expected):
            """Test subtraction with various inputs"""
            result = calc.subtract(a, b)
            assert result == expected
        
        @pytest.mark.parametrize("a,b,expected", [
            (3, 4, 12),
            (0, 5, 0),
            (-2, 3, -6),
            (2.5, 4, 10.0)
        ])
        def test_multiply(self, calc, a, b, expected):
            """Test multiplication with various inputs"""
            result = calc.multiply(a, b)
            assert result == expected
        
        @pytest.mark.parametrize("a,b,expected", [
            (10, 2, 5.0),
            (7, 2, 3.5),
            (0, 5, 0.0),
            (-6, 2, -3.0)
        ])
        def test_divide(self, calc, a, b, expected):
            """Test division with various inputs"""
            result = calc.divide(a, b)
            assert result == expected
        
        def test_divide_by_zero(self, calc):
            """Test division by zero raises ValueError"""
            with pytest.raises(ValueError, match="Cannot divide by zero"):
                calc.divide(10, 0)
            
            # Ensure no history entry was added
            assert len(calc.history) == 0
        
        @pytest.mark.parametrize("base,exp,expected", [
            (2, 3, 8),
            (5, 0, 1),
            (10, 2, 100),
            (3, 4, 81)
        ])
        def test_power(self, calc, base, exp, expected):
            """Test power calculation"""
            result = calc.power(base, exp)
            assert result == expected
        
        def test_power_negative_exponent(self, calc):
            """Test power with negative exponent raises ValueError"""
            with pytest.raises(ValueError, match="Negative exponents not supported"):
                calc.power(2, -3)
    
    # Memory functionality tests
    class TestMemoryFunctionality:
        """Test memory store/recall functionality"""
        
        def test_initial_memory(self, calc):
            """Test initial memory is zero"""
            assert calc.recall_memory() == 0
        
        def test_store_and_recall_memory(self, calc):
            """Test storing and recalling values"""
            calc.store_memory(42)
            assert calc.recall_memory() == 42
            
            calc.store_memory(-15.5)
            assert calc.recall_memory() == -15.5
        
        def test_clear_memory(self, calc):
            """Test clearing memory"""
            calc.store_memory(100)
            calc.clear_memory()
            assert calc.recall_memory() == 0
        
        def test_memory_operations_integration(self, calc):
            """Test using memory with calculations"""
            # Perform calculation and store result
            result = calc.add(10, 5)
            calc.store_memory(result)
            
            # Use memory in another calculation
            memory_value = calc.recall_memory()
            final_result = calc.multiply(memory_value, 2)
            
            assert final_result == 30
            assert len(calc.history) == 2
    
    # History functionality tests
    class TestHistoryFunctionality:
        """Test calculation history functionality"""
        
        def test_initial_history(self, calc):
            """Test initial history is empty"""
            assert calc.get_history() == []
        
        def test_history_tracking(self, calc):
            """Test that operations are tracked in history"""
            calc.add(2, 3)
            calc.subtract(10, 4)
            calc.multiply(3, 7)
            
            history = calc.get_history()
            assert len(history) == 3
            assert "2 + 3 = 5" in history
            assert "10 - 4 = 6" in history
            assert "3 * 7 = 21" in history
        
        def test_clear_history(self, calc):
            """Test clearing history"""
            calc.add(1, 1)
            calc.subtract(5, 2)
            
            assert len(calc.get_history()) == 2
            
            calc.clear_history()
            assert calc.get_history() == []
        
        def test_history_immutability(self, calc):
            """Test that returned history cannot modify internal history"""
            calc.add(1, 1)
            history = calc.get_history()
            history.append("fake entry")
            
            # Internal history should be unchanged
            assert len(calc.get_history()) == 1
            assert "fake entry" not in calc.get_history()
        
        def test_failed_operations_not_in_history(self, calc):
            """Test that failed operations don't appear in history"""
            try:
                calc.divide(10, 0)
            except ValueError:
                pass
            
            try:
                calc.power(2, -1)
            except ValueError:
                pass
            
            assert len(calc.get_history()) == 0
    
    # Integration tests
    class TestCalculatorIntegration:
        """Test complex scenarios and integration"""
        
        def test_complex_calculation_chain(self, calc):
            """Test a series of connected calculations"""
            # Calculate: ((10 + 5) * 3) / 5 ^ 2
            step1 = calc.add(10, 5)        # 15
            step2 = calc.multiply(step1, 3) # 45
            step3 = calc.power(5, 2)       # 25
            final = calc.divide(step2, step3) # 1.8
            
            assert final == 1.8
            assert len(calc.history) == 4
        
        def test_calculator_state_consistency(self, calc):
            """Test that calculator maintains consistent state"""
            # Perform operations
            calc.add(10, 20)
            calc.store_memory(100)
            calc.subtract(50, 25)
            
            # Verify state
            assert calc.recall_memory() == 100
            assert len(calc.get_history()) == 2
            
            # Clear memory but keep history
            calc.clear_memory()
            assert calc.recall_memory() == 0
            assert len(calc.get_history()) == 2
            
            # Clear history but memory should still be 0
            calc.clear_history()
            assert calc.recall_memory() == 0
            assert len(calc.get_history()) == 0
        
        @pytest.mark.parametrize("operations", [
            [(calc.add, 1, 2), (calc.multiply, 3, 4), (calc.subtract, 10, 5)],
            [(calc.divide, 20, 4), (calc.power, 2, 3), (calc.add, 1, 1)],
            [(calc.subtract, 100, 50), (calc.divide, 25, 5), (calc.multiply, 2, 3)]
        ])
        def test_multiple_operation_sequences(self, calc, operations):
            """Test various sequences of operations"""
            results = []
            for operation, a, b in operations:
                try:
                    result = operation(a, b)
                    results.append(result)
                except ValueError:
                    results.append(None)
            
            assert len(calc.get_history()) <= len(operations)
            assert len(results) == len(operations)
    
    # Performance and edge case tests
    class TestEdgeCases:
        """Test edge cases and boundary conditions"""
        
        def test_large_numbers(self, calc):
            """Test calculations with large numbers"""
            large_num = 10**10
            result = calc.add(large_num, large_num)
            assert result == 2 * large_num
        
        def test_very_small_numbers(self, calc):
            """Test calculations with very small numbers"""
            small_num = 0.0000001
            result = calc.multiply(small_num, 1000000)
            assert abs(result - 0.1) < 1e-10  # Account for floating point precision
        
        def test_floating_point_precision(self, calc):
            """Test floating point precision issues"""
            result = calc.add(0.1, 0.2)
            # Use pytest.approx for floating point comparisons
            assert result == pytest.approx(0.3)
        
        def test_zero_operations(self, calc):
            """Test operations involving zero"""
            assert calc
