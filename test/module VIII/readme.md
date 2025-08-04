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
