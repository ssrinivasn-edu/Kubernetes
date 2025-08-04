
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
