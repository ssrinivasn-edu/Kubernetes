# Module 7: Test Automation Frameworks (Pytest)
## 5-Day Python Programming Training - Day 3

**Duration:** 6 Hours  
**Prerequisites:** Python basics, Modules 1-6 completion  
**Learning Level:** Intermediate to Advanced

---

## Table of Contents
1. [Learning Objectives](#learning-objectives)
2. [Introduction to Test Automation](#introduction-to-test-automation)
3. [Setting Up Pytest Environment](#setting-up-pytest-environment)
4. [Writing Basic Test Cases](#writing-basic-test-cases)
5. [Test Discovery and Execution](#test-discovery-and-execution)
6. [Assertions and Test Organization](#assertions-and-test-organization)
7. [Fixtures Deep Dive](#fixtures-deep-dive)
8. [Parameterization](#parameterization)
9. [Mocking and Patching](#mocking-and-patching)
10. [Test Coverage and Reporting](#test-coverage-and-reporting)
11. [Advanced Pytest Features](#advanced-pytest-features)
12. [Real-World Testing Scenarios](#real-world-testing-scenarios)
13. [Best Practices](#best-practices)
14. [Summary and Next Steps](#summary-and-next-steps)

---

## Learning Objectives

By the end of this module, you will be able to:
- Understand the importance and principles of test automation
- Write comprehensive test cases using pytest framework
- Implement and use fixtures for test setup and teardown
- Use parameterization to test multiple scenarios efficiently
- Mock external dependencies and API calls
- Generate detailed test reports and coverage analysis
- Organize and structure test suites for large projects
- Apply testing best practices in professional development

---

## Introduction to Test Automation

### Why Test Automation?

Test automation is the practice of running tests automatically, managing test data, and utilizing test results to improve software quality. It's essential for:

- **Reliability**: Catch bugs before they reach production
- **Confidence**: Deploy with assurance that code works
- **Speed**: Run hundreds of tests in seconds
- **Consistency**: Same tests run the same way every time
- **Documentation**: Tests serve as living documentation

### Types of Testing

```
Testing Pyramid:
    /\
   /UI\     <- Few, Expensive, Slow
  /____\
 /  API \   <- More, Moderate Cost
/________\
/  UNIT   \  <- Many, Cheap, Fast
/__________\
```

1. **Unit Tests**: Test individual functions/methods
2. **Integration Tests**: Test component interactions
3. **API Tests**: Test service interfaces
4. **UI Tests**: Test user interface workflows

### Testing Principles

- **AAA Pattern**: Arrange, Act, Assert
- **FIRST**: Fast, Independent, Repeatable, Self-validating, Timely
- **Given-When-Then**: Behavior-driven development style

---

## Setting Up Pytest Environment

### Installation

```bash
# Install pytest and essential plugins
pip install pytest
pip install pytest-cov          # Coverage reporting
pip install pytest-html         # HTML reports
pip install pytest-xdist        # Parallel execution
pip install pytest-mock         # Enhanced mocking
pip install pytest-benchmark    # Performance testing
pip install pytest-asyncio      # Async testing
```

### Project Structure

```
project/
├── src/
│   ├── __init__.py
│   ├── calculator.py
│   ├── user_manager.py
│   └── api_client.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py           # Shared fixtures
│   ├── test_calculator.py
│   ├── test_user_manager.py
│   └── test_api_client.py
├── pytest.ini               # Configuration
└── requirements.txt
```

### Configuration (pytest.ini)

```ini
[tool:pytest]
# Test discovery patterns
testpaths = tests
python_files = test_*.py *_test.py
python_classes = Test*
python_functions = test_*

# Output options
addopts = 
    -v 
    --tb=short 
    --strict-markers
    --disable-warnings
    --cov=src
    --cov-report=html
    --cov-report=term-missing

# Custom markers
markers =
    slow: marks tests as slow
    integration: marks tests as integration tests
    unit: marks tests as unit tests
    api: marks tests as API tests
```

---

## Writing Basic Test Cases

### Sample Application Code

First, let's create some sample code to test:

```python
# src/calculator.py
class Calculator:
    """Simple calculator with basic operations"""
    
    def add(self, a, b):
        """Add two numbers"""
        if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
            raise TypeError("Arguments must be numbers")
        return a + b
    
    def subtract(self, a, b):
        """Subtract b from a"""
        if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
            raise TypeError("Arguments must be numbers")
        return a - b
    
    def multiply(self, a, b):
        """Multiply two numbers"""
        if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
            raise TypeError("Arguments must be numbers")
        return a * b
    
    def divide(self, a, b):
        """Divide a by b"""
        if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
            raise TypeError("Arguments must be numbers")
        if b == 0:
            raise ZeroDivisionError("Cannot divide by zero")
        return a / b
    
    def power(self, base, exponent):
        """Calculate base raised to exponent"""
        if not isinstance(base, (int, float)) or not isinstance(exponent, (int, float)):
            raise TypeError("Arguments must be numbers")
        return base ** exponent
```

### Basic Test Cases

```python
# tests/test_calculator.py
import pytest
from src.calculator import Calculator

class TestCalculator:
    """Test suite for Calculator class"""
    
    def setup_method(self):
        """Setup method called before each test"""
        self.calc = Calculator()
    
    def test_add_positive_numbers(self):
        """Test adding positive numbers"""
        # Arrange
        a, b = 5, 3
        expected = 8
        
        # Act
        result = self.calc.add(a, b)
        
        # Assert
        assert result == expected
    
    def test_add_negative_numbers(self):
        """Test adding negative numbers"""
        result = self.calc.add(-5, -3)
        assert result == -8
    
    def test_add_mixed_numbers(self):
        """Test adding positive and negative numbers"""
        result = self.calc.add(10, -3)
        assert result == 7
    
    def test_add_with_floats(self):
        """Test adding floating point numbers"""
        result = self.calc.add(1.5, 2.3)
        assert result == pytest.approx(3.8)  # Handle floating point precision
    
    def test_add_with_zero(self):
        """Test adding with zero"""
        assert self.calc.add(5, 0) == 5
        assert self.calc.add(0, 5) == 5
        assert self.calc.add(0, 0) == 0
    
    def test_subtract_basic(self):
        """Test basic subtraction"""
        assert self.calc.subtract(10, 3) == 7
        assert self.calc.subtract(5, 8) == -3
    
    def test_multiply_basic(self):
        """Test basic multiplication"""
        assert self.calc.multiply(4, 5) == 20
        assert self.calc.multiply(-3, 4) == -12
        assert self.calc.multiply(0, 100) == 0
    
    def test_divide_basic(self):
        """Test basic division"""
        assert self.calc.divide(10, 2) == 5
        assert self.calc.divide(7, 2) == 3.5
    
    def test_divide_by_zero_raises_exception(self):
        """Test that division by zero raises ZeroDivisionError"""
        with pytest.raises(ZeroDivisionError, match="Cannot divide by zero"):
            self.calc.divide(10, 0)
    
    def test_power_basic(self):
        """Test power calculation"""
        assert self.calc.power(2, 3) == 8
        assert self.calc.power(5, 0) == 1
        assert self.calc.power(2, -1) == 0.5
    
    def test_invalid_input_types(self):
        """Test that invalid input types raise TypeError"""
        with pytest.raises(TypeError, match="Arguments must be numbers"):
            self.calc.add("5", 3)
        
        with pytest.raises(TypeError, match="Arguments must be numbers"):
            self.calc.multiply(None, 5)
        
        with pytest.raises(TypeError, match="Arguments must be numbers"):
            self.calc.divide([1, 2], 3)
```

### Running Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_calculator.py

# Run specific test method
pytest tests/test_calculator.py::TestCalculator::test_add_positive_numbers

# Run with verbose output
pytest -v

# Run tests matching a pattern
pytest -k "add"
```

---

## Test Discovery and Execution

### Test Discovery Rules

Pytest automatically discovers tests based on these patterns:
- Files: `test_*.py` or `*_test.py`
- Classes: `Test*` (no `__init__` method)
- Functions: `test_*`

### Execution Options

```bash
# Stop after first failure
pytest -x

# Stop after N failures
pytest --maxfail=3

# Run tests in parallel
pytest -n auto

# Run only failed tests from last run
pytest --lf

# Run failed tests first, then remaining
pytest --ff

# Show local variables in tracebacks
pytest -l

# Capture output (show print statements)
pytest -s
```

### Markers and Test Selection

```python
# tests/test_advanced_calculator.py
import pytest
from src.calculator import Calculator

@pytest.mark.unit
def test_simple_addition():
    calc = Calculator()
    assert calc.add(2, 3) == 5

@pytest.mark.slow
def test_large_calculation():
    calc = Calculator()
    # Simulate a slow test
    import time
    time.sleep(0.1)
    assert calc.power(2, 1000) > 0

@pytest.mark.integration
def test_calculator_workflow():
    calc = Calculator()
    result = calc.add(10, 5)
    result = calc.multiply(result, 2)
    result = calc.subtract(result, 5)
    assert result == 25

@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    pass

@pytest.mark.xfail(reason="Known bug in power function")
def test_known_bug():
    calc = Calculator()
    # This test is expected to fail
    assert calc.power(-1, 0.5) > 0
```

```bash
# Run only unit tests
pytest -m unit

# Run all except slow tests
pytest -m "not slow"

# Run integration or API tests
pytest -m "integration or api"

# Show available markers
pytest --markers
```

---

## Assertions and Test Organization

### Pytest Assertion Introspection

Pytest provides detailed assertion failure messages:

```python
def test_assertion_examples():
    """Examples of different assertion types"""
    
    # Basic equality
    assert 2 + 2 == 4
    
    # Approximate equality for floats
    assert 0.1 + 0.2 == pytest.approx(0.3)
    
    # String operations
    result = "Hello World"
    assert "World" in result
    assert result.startswith("Hello")
    assert result.endswith("World")
    
    # List/Dict operations
    data = [1, 2, 3, 4, 5]
    assert len(data) == 5
    assert 3 in data
    assert data[0] == 1
    
    # Dictionary assertions
    user = {"name": "John", "age": 30}
    assert user["name"] == "John"
    assert "email" not in user
    
    # Boolean assertions
    assert True
    assert not False
    
    # None assertions
    value = None
    assert value is None
    
    # Type assertions
    assert isinstance("hello", str)
    assert isinstance([1, 2, 3], list)
```

### Custom Assertion Messages

```python
def test_custom_messages():
    """Test with custom assertion messages"""
    x = 10
    y = 5
    
    # Custom message for better debugging
    assert x > y, f"Expected {x} to be greater than {y}"
    
    # Multiple assertions with context
    username = "john_doe"
    assert len(username) >= 3, "Username too short"
    assert "_" in username, "Username should contain underscore"
    assert username.islower(), "Username should be lowercase"
```

### Test Organization Patterns

```python
# tests/test_user_manager.py
import pytest
from src.user_manager import UserManager, User

class TestUserManager:
    """Test suite for UserManager"""
    
    def setup_method(self):
        """Setup fresh UserManager for each test"""
        self.user_manager = UserManager()
    
    def teardown_method(self):
        """Cleanup after each test"""
        # Clean up resources if needed
        pass

class TestUserCreation:
    """Tests focused on user creation"""
    
    def test_create_user_with_valid_data(self):
        manager = UserManager()
        user = manager.create_user("john_doe", "john@email.com", 25)
        
        assert user.username == "john_doe"
        assert user.email == "john@email.com"
        assert user.age == 25
        assert user.id is not None
    
    def test_create_user_with_invalid_email(self):
        manager = UserManager()
        with pytest.raises(ValueError, match="Invalid email format"):
            manager.create_user("john", "invalid-email", 25)

class TestUserValidation:
    """Tests focused on user validation"""
    
    @pytest.mark.parametrize("email", [
        "invalid",
        "@domain.com",
        "user@",
        "user space@domain.com"
    ])
    def test_invalid_email_formats(self, email):
        manager = UserManager()
        with pytest.raises(ValueError):
            manager.create_user("user", email, 25)
```

---

## Fixtures Deep Dive

### What are Fixtures?

Fixtures are functions that provide test data, setup test environment, or perform cleanup. They promote code reuse and ensure consistent test conditions.

### Basic Fixtures

```python
# tests/conftest.py - Shared fixtures across all tests
import pytest
from src.calculator import Calculator
from src.user_manager import UserManager, User
import tempfile
import os

@pytest.fixture
def calculator():
    """Provide a Calculator instance for tests"""
    return Calculator()

@pytest.fixture
def user_manager():
    """Provide a UserManager instance for tests"""
    return UserManager()

@pytest.fixture
def sample_user():
    """Provide a sample user for tests"""
    return User("john_doe", "john@email.com", 25)

@pytest.fixture
def sample_users():
    """Provide multiple sample users"""
    users = [
        User("alice", "alice@email.com", 30),
        User("bob", "bob@email.com", 35),
        User("charlie", "charlie@email.com", 28)
    ]
    return users
```

### Fixture Scopes

```python
# Different fixture scopes control when fixtures are created/destroyed

@pytest.fixture(scope="function")  # Default - new instance per test
def function_scoped_data():
    print("Creating function-scoped fixture")
    return {"data": "function"}

@pytest.fixture(scope="class")  # One instance per test class
def class_scoped_data():
    print("Creating class-scoped fixture")
    return {"data": "class"}

@pytest.fixture(scope="module")  # One instance per test module
def module_scoped_data():
    print("Creating module-scoped fixture")
    return {"data": "module"}

@pytest.fixture(scope="session")  # One instance per test session
def session_scoped_data():
    print("Creating session-scoped fixture")
    return {"data": "session"}

# Database fixture example
@pytest.fixture(scope="session")
def database():
    """Create test database for entire session"""
    print("Setting up test database...")
    # Setup database
    db = create_test_database()
    yield db  # This provides the fixture value
    print("Tearing down test database...")
    # Cleanup database
    db.close()
```

### Fixture with Setup/Teardown

```python
@pytest.fixture
def temp_file():
    """Create a temporary file for testing"""
    # Setup
    fd, filepath = tempfile.mkstemp()
    os.write(fd, b"test data")
    os.close(fd)
    
    yield filepath  # Provide the filepath to test
    
    # Teardown
    os.unlink(filepath)

@pytest.fixture
def mock_api_server():
    """Start a mock API server for testing"""
    from unittest.mock import Mock
    import threading
    import time
    
    # Setup
    server = Mock()
    server.start()
    
    # Give server time to start
    time.sleep(0.1)
    
    yield server
    
    # Teardown
    server.stop()

# Using fixtures in tests
def test_file_operations(temp_file):
    """Test that uses temporary file fixture"""
    with open(temp_file, 'r') as f:
        content = f.read()
    assert content == "test data"

def test_api_call(mock_api_server):
    """Test that uses mock API server"""
    # Mock server is already running
    assert mock_api_server.is_running()
```

### Fixture Factories

```python
@pytest.fixture
def user_factory():
    """Factory fixture to create users with custom data"""
    def _create_user(username="default", email="default@email.com", age=25):
        return User(username, email, age)
    return _create_user

@pytest.fixture
def calculator_with_history():
    """Calculator fixture with operation history"""
    class CalculatorWithHistory(Calculator):
        def __init__(self):
            super().__init__()
            self.history = []
        
        def add(self, a, b):
            result = super().add(a, b)
            self.history.append(f"add({a}, {b}) = {result}")
            return result
    
    return CalculatorWithHistory()

# Using factory fixtures
def test_user_creation_factory(user_factory):
    """Test using user factory"""
    user1 = user_factory("alice", "alice@test.com", 30)
    user2 = user_factory("bob", "bob@test.com", 35)
    
    assert user1.username == "alice"
    assert user2.username == "bob"

def test_calculator_history(calculator_with_history):
    """Test calculator with history tracking"""
    calc = calculator_with_history
    calc.add(5, 3)
    calc.add(10, 2)
    
    assert len(calc.history) == 2
    assert "add(5, 3) = 8" in calc.history
```

### Advanced Fixture Patterns

```python
@pytest.fixture
def database_transaction():
    """Fixture that wraps each test in a database transaction"""
    from src.database import get_connection
    
    conn = get_connection()
    transaction = conn.begin()
    
    yield conn
    
    # Always rollback - tests don't affect each other
    transaction.rollback()
    conn.close()

@pytest.fixture(autouse=True)
def reset_singleton():
    """Auto-use fixture that resets singleton state"""
    yield  # Run the test
    
    # Reset any singleton state after each test
    from src.singleton_manager import SingletonManager
    SingletonManager.reset()

@pytest.fixture(params=["sqlite", "postgresql", "mysql"])
def database_type(request):
    """Parametrized fixture to test against multiple databases"""
    return request.param

def test_database_operations(database_type):
    """This test will run once for each database type"""
    # Test runs 3 times: once for sqlite, once for postgresql, once for mysql
    db = create_database(database_type)
    assert db.is_connected()
```

---

## Parameterization

### Basic Parameterization

```python
import pytest

@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (10, 5, 15),
    (-1, 1, 0),
    (0, 0, 0),
    (1.5, 2.5, 4.0)
])
def test_addition_multiple_cases(calculator, a, b, expected):
    """Test addition with multiple parameter sets"""
    result = calculator.add(a, b)
    assert result == expected

@pytest.mark.parametrize("operation,a,b,expected", [
    ("add", 5, 3, 8),
    ("subtract", 10, 4, 6),
    ("multiply", 3, 7, 21),
    ("divide", 15, 3, 5)
])
def test_calculator_operations(calculator, operation, a, b, expected):
    """Test multiple calculator operations"""
    method = getattr(calculator, operation)
    result = method(a, b)
    assert result == expected
```

### Advanced Parameterization

```python
# Named parameters for better test identification
@pytest.mark.parametrize("username,email,age,should_pass", [
    pytest.param("john_doe", "john@email.com", 25, True, id="valid_user"),
    pytest.param("", "john@email.com", 25, False, id="empty_username"),
    pytest.param("john_doe", "invalid-email", 25, False, id="invalid_email"),
    pytest.param("john_doe", "john@email.com", -5, False, id="negative_age"),
    pytest.param("a" * 100, "john@email.com", 25, False, id="username_too_long")
])
def test_user_validation(user_manager, username, email, age, should_pass):
    """Test user validation with various inputs"""
    if should_pass:
        user = user_manager.create_user(username, email, age)
        assert user is not None
    else:
        with pytest.raises(ValueError):
            user_manager.create_user(username, email, age)

# Indirect parameterization with fixtures
@pytest.fixture(params=[
    {"type": "admin", "permissions": ["read", "write", "delete"]},
    {"type": "user", "permissions": ["read"]},
    {"type": "guest", "permissions": []}
])
def user_type(request):
    return request.param

def test_user_permissions(user_type):
    """Test runs once for each user type"""
    user_info = user_type
    if user_info["type"] == "admin":
        assert "delete" in user_info["permissions"]
    elif user_info["type"] == "user":
        assert "read" in user_info["permissions"]
        assert "delete" not in user_info["permissions"]

# Multiple parameterizations
@pytest.mark.parametrize("browser", ["chrome", "firefox", "safari"])
@pytest.mark.parametrize("device", ["desktop", "tablet", "mobile"])
def test_web_app_compatibility(browser, device):
    """Test runs 9 times (3 browsers × 3 devices)"""
    # This creates a test matrix
    config = f"{browser}-{device}"
    # Test web app on different browser/device combinations
    assert True  # Placeholder test
```

### Dynamic Parameterization

```python
def get_test_data():
    """Dynamically generate test data"""
    return [
        (i, i * 2, i * 3) for i in range(1, 6)
    ]

@pytest.mark.parametrize("input,double,triple", get_test_data())
def test_dynamic_data(input, double, triple):
    """Test with dynamically generated data"""
    assert input * 2 == double
    assert input * 3 == triple

# Reading test data from external source
import json

def load_test_cases():
    """Load test cases from JSON file"""
    try:
        with open("test_data.json", "r") as f:
            return json.load(f)
    except FileNotFoundError:
        return [{"input": 1, "expected": 2}]  # Fallback data

@pytest.mark.parametrize("test_case", load_test_cases())
def test_from_external_data(calculator, test_case):
    """Test with data loaded from external file"""
    result = calculator.add(test_case["input"], test_case["input"])
    assert result == test_case["expected"]
```

---

## Mocking and Patching

### Why Mock?

Mocking allows you to:
- Isolate units under test
- Control external dependencies
- Test error conditions
- Speed up tests by avoiding slow operations
- Test without side effects

### Basic Mocking with unittest.mock

```python
from unittest.mock import Mock, patch, MagicMock
import pytest
import requests

# Sample code to test
# src/api_client.py
class APIClient:
    def __init__(self, base_url):
        self.base_url = base_url
    
    def get_user(self, user_id):
        """Get user data from API"""
        response = requests.get(f"{self.base_url}/users/{user_id}")
        if response.status_code == 200:
            return response.json()
        elif response.status_code == 404:
            return None
        else:
            raise Exception(f"API Error: {response.status_code}")
    
    def create_user(self, user_data):
        """Create a new user via API"""
        response = requests.post(
            f"{self.base_url}/users",
            json=user_data
        )
        if response.status_code == 201:
            return response.json()
        else:
            raise Exception(f"Failed to create user: {response.status_code}")

# Tests with mocking
class TestAPIClient:
    
    @patch('src.api_client.requests.get')
    def test_get_user_success(self, mock_get):
        """Test successful user retrieval"""
        # Arrange
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = {
            "id": 1,
            "name": "John Doe",
            "email": "john@email.com"
        }
        mock_get.return_value = mock_response
        
        client = APIClient("https://api.example.com")
        
        # Act
        user = client.get_user(1)
        
        # Assert
        assert user["name"] == "John Doe"
        mock_get.assert_called_once_with("https://api.example.com/users/1")
    
    @patch('src.api_client.requests.get')
    def test_get_user_not_found(self, mock_get):
        """Test user not found scenario"""
        # Arrange
        mock_response = Mock()
        mock_response.status_code = 404
        mock_get.return_value = mock_response
        
        client = APIClient("https://api.example.com")
        
        # Act
        user = client.get_user(999)
        
        # Assert
        assert user is None
        mock_get.assert_called_once_with("https://api.example.com/users/999")
    
    @patch('src.api_client.requests.get')
    def test_get_user_api_error(self, mock_get):
        """Test API error handling"""
        # Arrange
        mock_response = Mock()
        mock_response.status_code = 500
        mock_get.return_value = mock_response
        
        client = APIClient("https://api.example.com")
        
        # Act & Assert
        with pytest.raises(Exception, match="API Error: 500"):
            client.get_user(1)
```

### Context Manager Mocking

```python
def test_with_context_manager():
    """Test using mock as context manager"""
    client = APIClient("https://api.example.com")
    
    with patch('src.api_client.requests.get') as mock_get:
        # Setup mock
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = {"id": 1, "name": "Test User"}
        mock_get.return_value = mock_response
        
        # Test
        user = client.get_user(1)
        
        # Verify
        assert user["name"] == "Test User"
        mock_get.assert_called_once()
```

### Mock Object Attributes and Methods

```python
def test_mock_attributes():
    """Test mocking object attributes and methods"""
    
    # Create mock with predefined attributes
    mock_user = Mock()
    mock_user.name = "John Doe"
    mock_user.email = "john@email.com"
    mock_user.get_full_name.return_value = "John Doe"
    
    # Test
    assert mock_user.name == "John Doe"
    assert mock_user.get_full_name() == "John Doe"
    
    # Verify method was called
    mock_user.get_full_name.assert_called_once()

def test_mock_side_effects():
    """Test mock side effects"""
    
    # Mock that raises exception
    mock_function = Mock(side_effect=ValueError("Invalid input"))
    
    with pytest.raises(ValueError, match="Invalid input"):
        mock_function()
    
    # Mock with different return values on successive calls
    mock_function = Mock(side_effect=[1, 2, 3, StopIteration])
    
    assert mock_function() == 1
    assert mock_function() == 2
    assert mock_function() == 3
    
    with pytest.raises(StopIteration):
        mock_function()
```

### Advanced Mocking Patterns

```python
# src/file_processor.py
import os
import json

class FileProcessor:
    def process_config_file(self, filepath):
        """Process configuration file"""
        if not os.path.exists(filepath):
            raise FileNotFoundError(f"Config file not found: {filepath}")
        
        with open(filepath, 'r') as f:
            config = json.load(f)
        
        # Process configuration
        processed_config = {
            "database_url": config.get("db_url", "localhost"),
            "debug_mode": config.get("debug", False),
            "max_connections": config.get("max_conn", 10)
        }
        
        return processed_config

# Advanced mocking test
class TestFileProcessor:
    
    @patch('src.file_processor.os.path.exists')
    @patch('src.file_processor.open')
    @patch('src.file_processor.json.load')
    def test_process_config_file(self, mock_json_load, mock_open, mock_exists):
        """Test config file processing with multiple patches"""
        # Arrange
        mock_exists.return_value = True
        mock_json_load.return_value = {
            "db_url": "postgresql://localhost:5432/testdb",
            "debug": True,
            "max_conn": 20
        }
        
        processor = FileProcessor()
        
        # Act
        result = processor.process_config_file("config.json")
        
        # Assert
        assert result["database_url"] == "postgresql://localhost:5432/testdb"
        assert result["debug_mode"] is True
        assert result["max_connections"] == 20
        
        # Verify all mocks were called correctly
        mock_exists.assert_called_once_with("config.json")
        mock_open.assert_called_once_with("config.json", 'r')
        mock_json_load.assert_called_once()

    @patch('src.file_processor.os.path.exists', return_value=False)
    def test_process_config_file_not_found(self, mock_exists):
        """Test file not found scenario"""
        processor = FileProcessor()
        
        with pytest.raises(FileNotFoundError, match="Config file not found"):
            processor.process_config_file("missing.json")

# Pytest-mock plugin (cleaner syntax)
def test_with_pytest_mock(mocker):
    """Test using pytest-mock plugin"""
    # Mock requests.get
    mock_get = mocker.patch('requests.get')
    mock_response = mocker.Mock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"message": "success"}
    mock_get.return_value = mock_response
    
    client = APIClient("https://api.example.com")
    user = client.get_user(1)
    
    assert user["message"] == "success"

# Spy pattern - partial mocking
def test_spy_pattern(mocker):
    """Test spy pattern - monitor real method calls"""
    from src.calculator import Calculator
    
    calc = Calculator()
    
    # Spy on the add method
    spy_add = mocker.spy(calc, 'add')
    
    # Call the real method
    result = calc.add(5, 3)
    
    # Verify the real method was called and returned correct result
    assert result == 8
    spy_add.assert_called_once_with(5, 3)

# Mock class instances
@patch('src.api_client.APIClient')
def test_mock_class_instance(mock_api_client_class):
    """Test mocking entire class instances"""
    # Setup mock instance
    mock_instance = Mock()
    mock_instance.get_user.return_value = {"id": 1, "name": "Test"}
    mock_api_client_class.return_value = mock_instance
    
    # Test code that creates APIClient instance
    from src.user_service import UserService
    service = UserService("https://api.example.com")
    user = service.fetch_user_data(1)
    
    assert user["name"] == "Test"
    mock_api_client_class.assert_called_once_with("https://api.example.com")
```

---

## Test Coverage and Reporting

### Coverage Basics

Coverage measures how much of your code is executed during tests. Types include:
- **Line Coverage**: Percentage of code lines executed
- **Branch Coverage**: Percentage of code branches executed
- **Function Coverage**: Percentage of functions called

### Setting Up Coverage

```bash
# Install coverage tools
pip install pytest-cov coverage

# Run tests with coverage
pytest --cov=src
pytest --cov=src --cov-report=html
pytest --cov=src --cov-report=term-missing
pytest --cov=src --cov-report=xml

# Generate detailed HTML report
pytest --cov=src --cov-report=html --cov-report=term
```

### Coverage Configuration

```ini
# .coveragerc or setup.cfg
[run]
source = src
omit = 
    */tests/*
    */venv/*
    */virtualenv/*
    */__pycache__/*
    */site-packages/*

[report]
# Fail if coverage is below threshold
fail_under = 80

# Show missing lines
show_missing = True

# Exclude lines from coverage
exclude_lines =
    pragma: no cover
    def __repr__
    raise AssertionError
    raise NotImplementedError
    if __name__ == .__main__.:
```

### Coverage Examples

```python
# src/coverage_example.py
def complex_function(x, y, operation="add"):
    """Function with multiple branches for coverage testing"""
    
    if not isinstance(x, (int, float)):  # Branch 1
        raise TypeError("x must be a number")
    
    if not isinstance(y, (int, float)):  # Branch 2
        raise TypeError("y must be a number")
    
    if operation == "add":  # Branch 3
        return x + y
    elif operation == "subtract":  # Branch 4
        return x - y
    elif operation == "multiply":  # Branch 5
        return x * y
    elif operation == "divide":  # Branch 6
        if y == 0:  # Branch 7
            raise ZeroDivisionError("Cannot divide by zero")
        return x / y
    else:  # Branch 8
        raise ValueError(f"Unsupported operation: {operation}")

def rarely_used_function():  # pragma: no cover
    """This function is excluded from coverage"""
    return "This won't count against coverage"

# Comprehensive test for coverage
class TestCoverageExample:
    
    def test_add_operation(self):
        """Test addition - covers branches 3"""
        result = complex_function(5, 3, "add")
        assert result == 8
    
    def test_subtract_operation(self):
        """Test subtraction - covers branch 4"""
        result = complex_function(10, 3, "subtract")
        assert result == 7
    
    def test_multiply_operation(self):
        """Test multiplication - covers branch 5"""
        result = complex_function(4, 5, "multiply")
        assert result == 20
    
    def test_divide_operation(self):
        """Test division - covers branch 6"""
        result = complex_function(15, 3, "divide")
        assert result == 5
    
    def test_divide_by_zero(self):
        """Test division by zero - covers branch 7"""
        with pytest.raises(ZeroDivisionError):
            complex_function(10, 0, "divide")
    
    def test_invalid_x_type(self):
        """Test invalid x type - covers branch 1"""
        with pytest.raises(TypeError, match="x must be a number"):
            complex_function("invalid", 5)
    
    def test_invalid_y_type(self):
        """Test invalid y type - covers branch 2"""
        with pytest.raises(TypeError, match="y must be a number"):
            complex_function(5, "invalid")
    
    def test_invalid_operation(self):
        """Test invalid operation - covers branch 8"""
        with pytest.raises(ValueError, match="Unsupported operation"):
            complex_function(5, 3, "invalid")
    
    # This test achieves 100% branch coverage!
```

### HTML Coverage Reports

```python
# Generate and view HTML coverage report
# pytest --cov=src --cov-report=html

# This creates htmlcov/index.html with:
# - Overall coverage percentage
# - File-by-file coverage breakdown
# - Line-by-line coverage highlighting
# - Missing lines identification
```

---

## Advanced Pytest Features

### Custom Markers

```python
# conftest.py
import pytest

def pytest_configure(config):
    """Register custom markers"""
    config.addinivalue_line(
        "markers", "smoke: marks tests as smoke tests"
    )
    config.addinivalue_line(
        "markers", "regression: marks tests as regression tests"
    )
    config.addinivalue_line(
        "markers", "performance: marks tests as performance tests"
    )

# Using custom markers
@pytest.mark.smoke
def test_basic_functionality():
    """Basic smoke test"""
    assert True

@pytest.mark.regression
@pytest.mark.slow
def test_complex_scenario():
    """Regression test for complex scenario"""
    # Complex test logic
    pass

# Run specific marker tests
# pytest -m smoke
# pytest -m "smoke or regression"
# pytest -m "not slow"
```

### Plugins and Hooks

```python
# conftest.py - Custom hooks
import pytest
import time

def pytest_runtest_setup(item):
    """Hook called before each test"""
    print(f"\nSetting up test: {item.name}")

def pytest_runtest_teardown(item, nextitem):
    """Hook called after each test"""
    print(f"Tearing down test: {item.name}")

def pytest_runtest_call(pyfuncitem):
    """Hook called during test execution"""
    start_time = time.time()
    
    # Run the actual test
    outcome = pytest.PytestHooks.pytest_runtest_call.call_historic(
        kwargs=dict(pyfuncitem=pyfuncitem)
    )
    
    duration = time.time() - start_time
    if duration > 1.0:  # Log slow tests
        print(f"SLOW TEST: {pyfuncitem.name} took {duration:.2f}s")
    
    return outcome

# Custom collection hook
def pytest_collection_modifyitems(config, items):
    """Modify collected test items"""
    # Add markers based on test names
    for item in items:
        if "integration" in item.name:
            item.add_marker(pytest.mark.integration)
        if "slow" in item.name:
            item.add_marker(pytest.mark.slow)
```

### Temporary Directories and Files

```python
def test_with_tmp_path(tmp_path):
    """Test using temporary directory (pytest built-in fixture)"""
    # tmp_path is a pathlib.Path object
    test_file = tmp_path / "test.txt"
    test_file.write_text("Hello, World!")
    
    content = test_file.read_text()
    assert content == "Hello, World!"
    
    # Directory is automatically cleaned up

def test_with_tmpdir(tmpdir):
    """Test using temporary directory (legacy fixture)"""
    # tmpdir is a py.path.local object
    test_file = tmpdir.join("test.txt")
    test_file.write("Hello, World!")
    
    content = test_file.read()
    assert content == "Hello, World!"

# Custom temporary file fixture
@pytest.fixture
def config_file(tmp_path):
    """Create a temporary config file"""
    config_content = {
        "database": {
            "host": "localhost",
            "port": 5432,
            "name": "testdb"
        },
        "api": {
            "timeout": 30,
            "retries": 3
        }
    }
    
    config_path = tmp_path / "config.json"
    config_path.write_text(json.dumps(config_content, indent=2))
    
    return config_path

def test_config_loading(config_file):
    """Test configuration loading"""
    with open(config_file) as f:
        config = json.load(f)
    
    assert config["database"]["host"] == "localhost"
    assert config["api"]["timeout"] == 30
```

### Async Testing

```python
# src/async_api_client.py
import asyncio
import aiohttp

class AsyncAPIClient:
    def __init__(self, base_url):
        self.base_url = base_url
    
    async def get_user(self, user_id):
        """Async method to get user data"""
        async with aiohttp.ClientSession() as session:
            async with session.get(f"{self.base_url}/users/{user_id}") as response:
                if response.status == 200:
                    return await response.json()
                else:
                    raise Exception(f"API Error: {response.status}")

# Test async code
import pytest
import asyncio

@pytest.mark.asyncio
async def test_async_get_user():
    """Test async API client"""
    with patch('aiohttp.ClientSession.get') as mock_get:
        # Setup async mock
        mock_response = AsyncMock()
        mock_response.status = 200
        mock_response.json = AsyncMock(return_value={"id": 1, "name": "John"})
        
        mock_get.return_value.__aenter__.return_value = mock_response
        
        client = AsyncAPIClient("https://api.example.com")
        user = await client.get_user(1)
        
        assert user["name"] == "John"

# Async fixtures
@pytest.fixture
async def async_client():
    """Async fixture that sets up client"""
    client = AsyncAPIClient("https://api.example.com")
    yield client
    # Async cleanup if needed
    await client.close()  # If client has close method

@pytest.mark.asyncio
async def test_with_async_fixture(async_client):
    """Test using async fixture"""
    # Use the async client
    pass
```

---

## Real-World Testing Scenarios

### Testing API Integration

```python
# src/weather_service.py
import requests
from typing import Optional, Dict, Any

class WeatherService:
    def __init__(self, api_key: str, base_url: str = "http://api.openweathermap.org/data/2.5"):
        self.api_key = api_key
        self.base_url = base_url
    
    def get_current_weather(self, city: str) -> Optional[Dict[Any, Any]]:
        """Get current weather for a city"""
        try:
            params = {
                'q': city,
                'appid': self.api_key,
                'units': 'metric'
            }
            
            response = requests.get(f"{self.base_url}/weather", params=params)
            
            if response.status_code == 200:
                data = response.json()
                return {
                    'city': data['name'],
                    'temperature': data['main']['temp'],
                    'description': data['weather'][0]['description'],
                    'humidity': data['main']['humidity']
                }
            elif response.status_code == 404:
                return None
            else:
                raise Exception(f"Weather API error: {response.status_code}")
                
        except requests.RequestException as e:
            raise Exception(f"Network error: {str(e)}")
    
    def get_forecast(self, city: str, days: int = 5) -> list:
        """Get weather forecast"""
        try:
            params = {
                'q': city,
                'appid': self.api_key,
                'units': 'metric',
                'cnt': days * 8  # 8 forecasts per day (3-hour intervals)
            }
            
            response = requests.get(f"{self.base_url}/forecast", params=params)
            
            if response.status_code == 200:
                data = response.json()
                forecasts = []
                
                for item in data['list']:
                    forecasts.append({
                        'datetime': item['dt_txt'],
                        'temperature': item['main']['temp'],
                        'description': item['weather'][0]['description']
                    })
                
                return forecasts
            else:
                raise Exception(f"Forecast API error: {response.status_code}")
                
        except requests.RequestException as e:
            raise Exception(f"Network error: {str(e)}")

# Comprehensive test suite
class TestWeatherService:
    
    @pytest.fixture
    def weather_service(self):
        """Fixture providing WeatherService instance"""
        return WeatherService("test_api_key")
    
    @pytest.fixture
    def mock_weather_response(self):
        """Fixture with sample weather API response"""
        return {
            "name": "London",
            "main": {
                "temp": 15.5,
                "humidity": 80
            },
            "weather": [
                {
                    "description": "scattered clouds"
                }
            ]
        }
    
    @pytest.fixture
    def mock_forecast_response(self):
        """Fixture with sample forecast API response"""
        return {
            "list": [
                {
                    "dt_txt": "2023-12-01 12:00:00",
                    "main": {"temp": 16.0},
                    "weather": [{"description": "clear sky"}]
                },
                {
                    "dt_txt": "2023-12-01 15:00:00",
                    "main": {"temp": 18.5},
                    "weather": [{"description": "few clouds"}]
                }
            ]
        }
    
    @patch('src.weather_service.requests.get')
    def test_get_current_weather_success(self, mock_get, weather_service, mock_weather_response):
        """Test successful weather retrieval"""
        # Arrange
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_weather_response
        mock_get.return_value = mock_response
        
        # Act
        weather = weather_service.get_current_weather("London")
        
        # Assert
        assert weather is not None
        assert weather['city'] == "London"
        assert weather['temperature'] == 15.5
        assert weather['description'] == "scattered clouds"
        assert weather['humidity'] == 80
        
        # Verify API call
        mock_get.assert_called_once()
        call_args = mock_get.call_args
        assert "weather" in call_args[0][0]
        assert call_args[1]['params']['q'] == "London"
        assert call_args[1]['params']['appid'] == "test_api_key"
    
    @patch('src.weather_service.requests.get')
    def test_get_current_weather_city_not_found(self, mock_get, weather_service):
        """Test city not found scenario"""
        # Arrange
        mock_response = Mock()
        mock_response.status_code = 404
        mock_get.return_value = mock_response
        
        # Act
        weather = weather_service.get_current_weather("NonexistentCity")
        
        # Assert
        assert weather is None
    
    @patch('src.weather_service.requests.get')
    def test_get_current_weather_api_error(self, mock_get, weather_service):
        """Test API error handling"""
        # Arrange
        mock_response = Mock()
        mock_response.status_code = 500
        mock_get.return_value = mock_response
        
        # Act & Assert
        with pytest.raises(Exception, match="Weather API error: 500"):
            weather_service.get_current_weather("London")
    
    @patch('src.weather_service.requests.get')
    def test_get_current_weather_network_error(self, mock_get, weather_service):
        """Test network error handling"""
        # Arrange
        mock_get.side_effect = requests.RequestException("Connection timeout")
        
        # Act & Assert
        with pytest.raises(Exception, match="Network error: Connection timeout"):
            weather_service.get_current_weather("London")
    
    @patch('src.weather_service.requests.get')
    def test_get_forecast_success(self, mock_get, weather_service, mock_forecast_response):
        """Test successful forecast retrieval"""
        # Arrange
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_forecast_response
        mock_get.return_value = mock_response
        
        # Act
        forecast = weather_service.get_forecast("London", days=1)
        
        # Assert
        assert len(forecast) == 2
        assert forecast[0]['datetime'] == "2023-12-01 12:00:00"
        assert forecast[0]['temperature'] == 16.0
        assert forecast[1]['temperature'] == 18.5
        
        # Verify API call
        call_args = mock_get.call_args
        assert call_args[1]['params']['cnt'] == 8  # 1 day * 8 forecasts
    
    @pytest.mark.parametrize("city,expected_calls", [
        ("London", 1),
        ("New York", 1),
        ("Tokyo", 1)
    ])
    @patch('src.weather_service.requests.get')
    def test_multiple_cities(self, mock_get, weather_service, mock_weather_response, city, expected_calls):
        """Test weather service with multiple cities"""
        # Arrange
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_weather_response
        mock_get.return_value = mock_response
        
        # Act
        weather = weather_service.get_current_weather(city)
        
        # Assert
        assert weather is not None
        assert mock_get.call_count == expected_calls
```

### Testing Database Operations

```python
# src/user_repository.py
import sqlite3
from typing import List, Optional

class User:
    def __init__(self, user_id: int, username: str, email: str, age: int):
        self.id = user_id
        self.username = username
        self.email = email
        self.age = age
    
    def to_dict(self):
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
            'age': self.age
        }

class UserRepository:
    def __init__(self, db_path: str):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        """Initialize database schema"""
        with sqlite3.connect(self.db_path) as conn:
            conn.execute('''
                CREATE TABLE IF NOT EXISTS users (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    username TEXT UNIQUE NOT NULL,
                    email TEXT UNIQUE NOT NULL,
                    age INTEGER NOT NULL
                )
            ''')
            conn.commit()
    
    def create_user(self, username: str, email: str, age: int) -> User:
        """Create a new user"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                'INSERT INTO users (username, email, age) VALUES (?, ?, ?)',
                (username, email, age)
            )
            user_id = cursor.lastrowid
            conn.commit()
            
            return User(user_id, username, email, age)
    
    def get_user(self, user_id: int) -> Optional[User]:
        """Get user by ID"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                'SELECT id, username, email, age FROM users WHERE id = ?',
                (user_id,)
            )
            row = cursor.fetchone()
            
            if row:
                return User(*row)
            return None
    
    def get_all_users(self) -> List[User]:
        """Get all users"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                'SELECT id, username, email, age FROM users ORDER BY username'
            )
            rows = cursor.fetchall()
            
            return [User(*row) for row in rows]
    
    def update_user(self, user_id: int, **kwargs) -> bool:
        """Update user fields"""
        allowed_fields = ['username', 'email', 'age']
        updates = {k: v for k, v in kwargs.items() if k in allowed_fields}
        
        if not updates:
            return False
        
        set_clause = ', '.join([f"{field} = ?" for field in updates.keys()])
        values = list(updates.values()) + [user_id]
        
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                f'UPDATE users SET {set_clause} WHERE id = ?',
                values
            )
            conn.commit()
            
            return cursor.rowcount > 0
    
    def delete_user(self, user_id: int) -> bool:
        """Delete user by ID"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute('DELETE FROM users WHERE id = ?', (user_id,))
            conn.commit()
            
            return cursor.rowcount > 0

# Database testing
class TestUserRepository:
    
    @pytest.fixture
    def db_path(self, tmp_path):
        """Provide temporary database path"""
        return tmp_path / "test.db"
    
    @pytest.fixture
    def user_repo(self, db_path):
        """Provide UserRepository instance with clean database"""
        return UserRepository(str(db_path))
    
    @pytest.fixture
    def sample_users(self, user_repo):
        """Create sample users for testing"""
        users = []
        users.append(user_repo.create_user("alice", "alice@test.com", 25))
        users.append(user_repo.create_user("bob", "bob@test.com", 30))
        users.append(user_repo.create_user("charlie", "charlie@test.com", 35))
        return users
    
    def test_create_user(self, user_repo):
        """Test user creation"""
        user = user_repo.create_user("john", "john@test.com", 28)
        
        assert user.id is not None
        assert user.username == "john"
        assert user.email == "john@test.com"
        assert user.age == 28
    
    def test_get_user_exists(self, user_repo):
        """Test getting existing user"""
        # Create user first
        created_user = user_repo.create_user("jane", "jane@test.com", 26)
        
        # Retrieve user
        retrieved_user = user_repo.get_user(created_user.id)
        
        assert retrieved_user is not None
        assert retrieved_user.id == created_user.id
        assert retrieved_user.username == "jane"
        assert retrieved_user.email == "jane@test.com"
        assert retrieved_user.age == 26
    
    def test_get_user_not_exists(self, user_repo):
        """Test getting non-existent user"""
        user = user_repo.get_user(999)
        assert user is None
    
    def test_get_all_users(self, user_repo, sample_users):
        """Test getting all users"""
        all_users = user_repo.get_all_users()
        
        assert len(all_users) == 3
        usernames = [user.username for user in all_users]
        assert "alice" in usernames
        assert "bob" in usernames
        assert "charlie" in usernames
    
    def test_update_user_success(self, user_repo):
        """Test successful user update"""
        user = user_repo.create_user("test", "test@test.com", 20)
        
        success = user_repo.update_user(user.id, username="updated", age=25)
        assert success is True
        
        updated_user = user_repo.get_user(user.id)
        assert updated_user.username == "updated"
        assert updated_user.age == 25
        assert updated_user.email == "test@test.com"  # Unchanged
    
    def test_update_user_not_exists(self, user_repo):
        """Test updating non-existent user"""
        success = user_repo.update_user(999, username="test")
        assert success is False
    
    def test_delete_user_success(self, user_repo):
        """Test successful user deletion"""
        user = user_repo.create_user("delete_me", "delete@test.com", 22)
        
        success = user_repo.delete_user(user.id)
        assert success is True
        
        deleted_user = user_repo.get_user(user.id)
        assert deleted_user is None
    
    def test_delete_user_not_exists(self, user_repo):
        """Test deleting non-existent user"""
        success = user_repo.delete_user(999)
        assert success is False
    
    def test_unique_constraints(self, user_repo):
        """Test database unique constraints"""
        user_repo.create_user("unique", "unique@test.com", 30)
        
        # Try to create user with same username
        with pytest.raises(sqlite3.IntegrityError):
            user_repo.create_user("unique", "different@test.com", 25)
        
        # Try to create user with same email
        with pytest.raises(sqlite3.IntegrityError):
            user_repo.create_user("different", "unique@test.com", 25)
    
    @pytest.mark.parametrize("username,email,age", [
        ("user1", "user1@test.com", 20),
        ("user2", "user2@test.com", 30),
        ("user3", "user3@test.com", 40)
    ])
    def test_create_multiple_users(self, user_repo, username, email, age):
        """Test creating multiple users with parameterization"""
        user = user_repo.create_user(username, email, age)
        assert user.username == username
        assert user.email == email
        assert user.age == age
```

---

## Best Practices

### Test Organization and Structure

```python
# Good test organization
class TestUserAuthentication:
    """Group related authentication tests"""
    
    def test_login_with_valid_credentials(self):
        pass
    
    def test_login_with_invalid_password(self):
        pass
    
    def test_login_with_nonexistent_user(self):
        pass

class TestUserRegistration:
    """Group related registration tests"""
    
    def test_register_with_valid_data(self):
        pass
    
    def test_register_with_duplicate_email(self):
        pass

# Use descriptive test names
def test_should_return_user_data_when_valid_id_provided():
    """Clear, descriptive test name"""
    pass

def test_should_raise_value_error_when_negative_age_provided():
    """Describes expected behavior clearly"""
    pass
```

### Test Data Management

```python
# conftest.py - Centralized test data
@pytest.fixture
def valid_user_data():
    """Standard valid user data for tests"""
    return {
        "username": "testuser",
        "email": "test@example.com",
        "age": 25,
        "preferences": {
            "theme": "dark",
            "notifications": True
        }
    }

@pytest.fixture
def invalid_user_data_samples():
    """Collection of invalid user data for testing"""
    return [
        {"username": "", "email": "test@example.com", "age": 25},  # Empty username
        {"username": "test", "email": "invalid-email", "age": 25},  # Invalid email
        {"username": "test", "email": "test@example.com", "age": -5},  # Invalid age
        {"username": None, "email": "test@example.com", "age": 25},  # None username
    ]

# Test data factories
class UserDataFactory:
    """Factory for creating test user data"""
    
    @staticmethod
    def create_valid_user(**overrides):
        """Create valid user data with optional overrides"""
        default_data = {
            "username": "testuser",
            "email": "test@example.com",
            "age": 25
        }
        default_data.update(overrides)
        return default_data
    
    @staticmethod
    def create_admin_user(**overrides):
        """Create admin user data"""
        admin_data = {
            "username": "admin",
            "email": "admin@example.com",
            "age": 35,
            "role": "admin",
            "permissions": ["read", "write", "delete"]
        }
        admin_data.update(overrides)
        return admin_data

# Usage in tests
def test_user_creation_with_factory():
    user_data = UserDataFactory.create_valid_user(username="custom_user")
    assert user_data["username"] == "custom_user"
    assert user_data["email"] == "test@example.com"  # Default value
```

### Test Independence and Isolation

```python
class TestUserService:
    """Example of proper test isolation"""
    
    def setup_method(self):
        """Setup fresh state before each test"""
        self.user_service = UserService()
        self.test_users = []
    
    def teardown_method(self):
        """Cleanup after each test"""
        # Clean up any created test users
        for user in self.test_users:
            try:
                self.user_service.delete_user(user.id)
            except:
                pass  # User might already be deleted
    
    def test_create_user_independent(self):
        """Each test is independent"""
        user = self.user_service.create_user("test1", "test1@example.com")
        self.test_users.append(user)  # Track for cleanup
        assert user.username == "test1"
    
    def test_delete_user_independent(self):
        """This test doesn't depend on previous test"""
        user = self.user_service.create_user("test2", "test2@example.com")
        self.test_users.append(user)
        
        success = self.user_service.delete_user(user.id)
        assert success is True

# Avoid shared mutable state
class TestBadExample:
    """Example of what NOT to do"""
    
    shared_data = []  # BAD: Shared mutable state
    
    def test_append_data(self):
        self.shared_data.append("test1")  # BAD: Modifies shared state
        assert len(self.shared_data) == 1
    
    def test_more_data(self):
        self.shared_data.append("test2")  # BAD: Depends on previous test
        assert len(self.shared_data) == 2  # This will fail if run in isolation

class TestGoodExample:
    """Example of proper test isolation"""
    
    def test_append_data(self):
        data = []  # GOOD: Fresh data for each test
        data.append("test1")
        assert len(data) == 1
    
    def test_more_data(self):
        data = []  # GOOD: Independent test data
        data.extend(["test1", "test2"])
        assert len(data) == 2
```

### Error Testing Patterns

```python
class TestErrorHandling:
    """Comprehensive error testing patterns"""
    
    def test_exception_type_and_message(self):
        """Test specific exception type and message"""
        calculator = Calculator()
        
        with pytest.raises(ZeroDivisionError, match="Cannot divide by zero"):
            calculator.divide(10, 0)
    
    def test_exception_with_regex_match(self):
        """Test exception message with regex"""
        user_service = UserService()
        
        with pytest.raises(ValueError, match=r"Age must be between \d+ and \d+"):
            user_service.create_user("test", "test@example.com", -5)
    
    def test_multiple_possible_exceptions(self):
        """Test when multiple exceptions are possible"""
        api_client = APIClient("https://api.example.com")
        
        with pytest.raises((ConnectionError, TimeoutError, RequestException)):
            api_client.get_data_with_network_issues()
    
    def test_exception_context_manager(self):
        """Test exception details using context manager"""
        calculator = Calculator()
        
        with pytest.raises(TypeError) as exc_info:
            calculator.add("5", 3)
        
        # Access exception details
        assert "Arguments must be numbers" in str(exc_info.value)
        assert exc_info.type == TypeError
    
    def test_no_exception_raised(self):
        """Test that no exception is raised in valid case"""
        calculator = Calculator()
        
        # This should not raise any exception
        result = calculator.add(5, 3)
        assert result == 8
    
    @pytest.mark.parametrize("invalid_input,expected_exception", [
        ("string", TypeError),
        (None, TypeError),
        ([], TypeError),
        ({}, TypeError)
    ])
    def test_various_invalid_inputs(self, invalid_input, expected_exception):
        """Test various invalid inputs raise expected exceptions"""
        calculator = Calculator()
        
        with pytest.raises(expected_exception):
            calculator.add(invalid_input, 5)
```

### Performance and Load Testing

```python
import time
import pytest
from concurrent.futures import ThreadPoolExecutor, as_completed

class TestPerformance:
    """Performance testing examples"""
    
    @pytest.mark.performance
    def test_function_performance(self):
        """Test function execution time"""
        calculator = Calculator()
        
        start_time = time.time()
        
        # Perform operation many times
        for i in range(10000):
            calculator.add(i, i + 1)
        
        execution_time = time.time() - start_time
        
        # Assert performance requirement
        assert execution_time < 1.0, f"Function too slow: {execution_time:.2f}s"
    
    @pytest.mark.performance
    def test_memory_usage(self):
        """Test memory usage (requires psutil)"""
        import psutil
        import os
        
        process = psutil.Process(os.getpid())
        initial_memory = process.memory_info().rss
        
        # Perform memory-intensive operation
        large_list = [i for i in range(100000)]
        
        current_memory = process.memory_info().rss
        memory_increase = current_memory - initial_memory
        
        # Assert memory usage is reasonable (in bytes)
        assert memory_increase < 50 * 1024 * 1024, "Memory usage too high"
        
        # Cleanup
        del large_list
    
    @pytest.mark.slow
    def test_concurrent_operations(self):
        """Test concurrent access to shared resource"""
        user_service = UserService()
        results = []
        
        def create_user_worker(user_id):
            try:
                user = user_service.create_user(f"user_{user_id}", f"user_{user_id}@test.com")
                return user.id
            except Exception as e:
                return str(e)
        
        # Test with multiple threads
        with ThreadPoolExecutor(max_workers=10) as executor:
            futures = [executor.submit(create_user_worker, i) for i in range(50)]
            
            for future in as_completed(futures):
                results.append(future.result())
        
        # Verify results
        successful_creates = [r for r in results if isinstance(r, int)]
        assert len(successful_creates) >= 45, "Too many concurrent failures"
    
    @pytest.mark.benchmark
    def test_with_pytest_benchmark(self, benchmark):
        """Test using pytest-benchmark plugin"""
        calculator = Calculator()
        
        # Benchmark the add function
        result = benchmark(calculator.add, 100, 200)
        assert result == 300
        
        # The benchmark plugin will automatically measure and report performance
```

### Test Documentation and Reporting

```python
class TestDocumentationExamples:
    """Examples of well-documented tests"""
    
    def test_user_registration_workflow(self):
        """
        Test the complete user registration workflow.
        
        This test verifies that:
        1. A new user can be registered with valid data
        2. The user receives a confirmation email
        3. The user can activate their account
        4. The user can log in after activation
        
        Given: Valid user registration data
        When: User submits registration form
        Then: User account is created and activation email is sent
        """
        # Test implementation here
        pass
    
    def test_password_validation_rules(self):
        """
        Test password validation according to security policy.
        
        Password requirements:
        - Minimum 8 characters
        - At least 1 uppercase letter
        - At least 1 lowercase letter  
        - At least 1 digit
        - At least 1 special character
        
        Test covers both positive and negative cases.
        """
        # Test implementation here
        pass

# Custom test report generation
def pytest_html_report_title(report):
    """Customize HTML report title"""
    report.title = "User Management System Test Report"

def pytest_html_results_summary(prefix, summary, postfix):
    """Add custom summary to HTML report"""
    prefix.extend([
        "<h2>Test Environment</h2>",
        "<p>Python Version: 3.9.7</p>",
        "<p>Test Database: SQLite (in-memory)</p>",
        "<p>Mock API Server: Enabled</p>"
    ])

# Generate custom test report
class TestReportGenerator:
    """Generate custom test reports"""
    
    @classmethod
    def generate_coverage_summary(cls):
        """Generate coverage summary"""
        return {
            "total_lines": 1000,
            "covered_lines": 850,
            "coverage_percentage": 85.0,
            "missing_lines": ["user_service.py:45-50", "api_client.py:123"]
        }
    
    @classmethod
    def generate_performance_report(cls, test_results):
        """Generate performance test report"""
        slow_tests = [
            test for test in test_results 
            if test.get('duration', 0) > 1.0
        ]
        
        return {
            "total_tests": len(test_results),
            "slow_tests": len(slow_tests),
            "average_duration": sum(t.get('duration', 0) for t in test_results) / len(test_results),
            "slowest_test": max(test_results, key=lambda t: t.get('duration', 0))
        }
```

---

## Summary and Next Steps

### Key Concepts Mastered

✅ **Test Automation Fundamentals**
- Understanding of testing pyramid and test types
- Test-driven development principles
- AAA pattern (Arrange, Act, Assert)

✅ **Pytest Framework Mastery**
- Test discovery and execution
- Assertion introspection and custom messages
- Test organization and naming conventions

✅ **Fixtures and Test Setup**
- Fixture scopes and lifecycle management
- Setup/teardown patterns
- Fixture factories and parameterization

✅ **Advanced Testing Techniques**
- Mocking and patching external dependencies
- Testing async code and database operations
- Error handling and exception testing

✅ **Test Quality and Maintenance**
- Code coverage measurement and reporting
- Performance and load testing
- Test independence and isolation

✅ **Professional Testing Practices**
- CI/CD integration patterns
- Test documentation and reporting
- Debugging and troubleshooting tests

### Practical Skills Acquired

1. **Writing Comprehensive Test Suites** - Can create thorough test coverage for any Python application
2. **Mocking Complex Dependencies** - Can isolate units under test from external systems
3. **Test Data Management** - Can create maintainable test data and fixtures
4. **Performance Testing** - Can measure and verify application performance requirements
5. **Test Automation Integration** - Can integrate tests into development workflows

### Real-World Applications

- **API Testing**: Comprehensive testing of REST APIs with various scenarios
- **Database Testing**: Testing CRUD operations with proper isolation
- **Integration Testing**: Testing component interactions and workflows  
- **Error Handling**: Ensuring robust error handling and edge cases
- **Performance Validation**: Verifying system performance requirements

### Next Module Preview

**Module 9: Working with Excel and PDFs** (Day 3 continuation) - You'll learn to:
- Read and write Excel files using `openpyxl`
- Manipulate PDF documents with `PyPDF2` and `pdfplumber`
- Automate report generation workflows
- Extract and process document data
- Create business document automation solutions

### Practice Assignments

Before continuing, practice these exercises:

1. **Calculator Test Suite**: Complete the calculator test suite with 100% coverage
2. **Mock API Testing**: Create tests for the WeatherService with comprehensive mocking
3. **Database Testing**: Build a complete test suite for a user management system
4. **Performance Testing**: Add performance tests to verify response times
5. **Integration Testing**: Create end-to-end workflow tests

### Testing Checklist for Your Projects

- [ ] All functions have corresponding unit tests
- [ ] Edge cases and error conditions are tested
- [ ] External dependencies are properly mocked
- [ ] Test coverage is above 80%
- [ ] Tests are independent and can run in any order
- [ ] Test names clearly describe what is being tested
- [ ] Fixtures are used for common test setup
- [ ] Performance requirements are validated
- [ ] Integration tests cover key workflows

---

## Additional Resources

### Essential Reading
- [Pytest Documentation](https://docs.pytest.org/)
- [Python Testing 101](https://realpython.com/python-testing/)
- [Test-Driven Development with Python](https://www.obeythetestinggoat.com/)

### Advanced Topics
- [Property-Based Testing with Hypothesis](https://hypothesis.readthedocs.io/)
- [Mutation Testing with Mutmut](https://github.com/boxed/mutmut)
- [BDD Testing with Pytest-BDD](https://pytest-bdd.readthedocs.io/)

### Tools and Plugins
- [pytest-xdist](https://pytest-xdist.readthedocs.io/) - Parallel test execution
- [pytest-mock](https://pytest-mock.readthedocs.io/) - Enhanced mocking
- [pytest-cov](https://pytest-cov.readthedocs.io/) - Coverage reporting
- [pytest-html](https://pytest-html.readthedocs.io/) - HTML test reports
- [pytest-benchmark](https://pytest-benchmark.readthedocs.io/) - Performance testing

---

**Next Session:** Module 9 - Working with Excel and PDFs - 2 Hours  
**Preparation:** Review today's mocking examples and practice writing tests for your own code
