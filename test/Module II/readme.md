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

