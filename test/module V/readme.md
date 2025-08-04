
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
