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
