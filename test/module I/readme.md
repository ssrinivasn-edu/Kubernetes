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

