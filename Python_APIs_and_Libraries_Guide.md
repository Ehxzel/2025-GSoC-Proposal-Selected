# Working with APIs and Libraries in Python

A comprehensive guide to making HTTP requests, parsing JSON, managing dependencies, and working with external packages in Python.

---

## Table of Contents

1. [Introduction to APIs](#introduction-to-apis)
2. [Making HTTP Requests](#making-http-requests)
3. [Parsing JSON Data](#parsing-json-data)
4. [Working with External Packages](#working-with-external-packages)
5. [Virtual Environments](#virtual-environments)
6. [Dependency Management](#dependency-management)
7. [Best Practices](#best-practices)
8. [Practical Examples](#practical-examples)

---

## Introduction to APIs

### What is an API?

An **API (Application Programming Interface)** is a set of rules and protocols that allows different software applications to communicate with each other. In web development, APIs typically enable your application to request and receive data from external services.

### Types of APIs

- **REST APIs**: Use HTTP methods (GET, POST, PUT, DELETE) to interact with resources
- **GraphQL APIs**: Query language for APIs that allows clients to request specific data
- **SOAP APIs**: Protocol-based APIs using XML for message format
- **WebSocket APIs**: Enable real-time, bidirectional communication

### HTTP Methods

- **GET**: Retrieve data from a server
- **POST**: Send data to create a new resource
- **PUT**: Update an existing resource
- **DELETE**: Remove a resource
- **PATCH**: Partially update a resource

---

## Making HTTP Requests

### Using the `requests` Library

The `requests` library is the most popular and user-friendly way to make HTTP requests in Python.

#### Installation

```bash
pip install requests
```

#### Basic GET Request

```python
import requests

# Make a GET request
response = requests.get('https://api.github.com/users/octocat')

# Check if request was successful
if response.status_code == 200:
    print("Success!")
    print(response.text)
else:
    print(f"Error: {response.status_code}")
```

#### Request with Parameters

```python
import requests

# Add query parameters
params = {
    'q': 'python',
    'sort': 'stars',
    'order': 'desc'
}

response = requests.get('https://api.github.com/search/repositories', params=params)
data = response.json()
print(f"Found {data['total_count']} repositories")
```

#### POST Request with JSON Data

```python
import requests

url = 'https://jsonplaceholder.typicode.com/posts'
data = {
    'title': 'My New Post',
    'body': 'This is the content',
    'userId': 1
}

response = requests.post(url, json=data)
print(f"Status Code: {response.status_code}")
print(f"Response: {response.json()}")
```

#### Adding Headers

```python
import requests

headers = {
    'Authorization': 'Bearer YOUR_ACCESS_TOKEN',
    'Content-Type': 'application/json',
    'User-Agent': 'MyApp/1.0'
}

response = requests.get('https://api.example.com/data', headers=headers)
```

#### Handling Timeouts

```python
import requests

try:
    response = requests.get('https://api.example.com/data', timeout=5)
    response.raise_for_status()  # Raises HTTPError for bad responses
except requests.Timeout:
    print("Request timed out")
except requests.HTTPError as e:
    print(f"HTTP Error: {e}")
except requests.RequestException as e:
    print(f"Error: {e}")
```

#### Session Objects for Multiple Requests

```python
import requests

# Create a session to persist parameters across requests
session = requests.Session()
session.headers.update({'Authorization': 'Bearer TOKEN'})

# All requests using this session will include the header
response1 = session.get('https://api.example.com/endpoint1')
response2 = session.get('https://api.example.com/endpoint2')
```

### Using `urllib` (Standard Library)

For simple requests without external dependencies:

```python
import urllib.request
import urllib.parse

# GET request
response = urllib.request.urlopen('https://api.github.com/users/octocat')
data = response.read()
print(data.decode('utf-8'))

# POST request
url = 'https://httpbin.org/post'
values = {'name': 'John', 'email': 'john@example.com'}
data = urllib.parse.urlencode(values).encode('utf-8')
req = urllib.request.Request(url, data)
response = urllib.request.urlopen(req)
print(response.read().decode('utf-8'))
```

### Async HTTP Requests with `aiohttp`

For concurrent requests in asynchronous applications:

```python
import aiohttp
import asyncio

async def fetch_data(session, url):
    async with session.get(url) as response:
        return await response.json()

async def main():
    async with aiohttp.ClientSession() as session:
        urls = [
            'https://api.github.com/users/user1',
            'https://api.github.com/users/user2',
            'https://api.github.com/users/user3'
        ]

        tasks = [fetch_data(session, url) for url in urls]
        results = await asyncio.gather(*tasks)

        for result in results:
            print(result['login'])

# Run the async function
asyncio.run(main())
```

---

## Parsing JSON Data

### What is JSON?

**JSON (JavaScript Object Notation)** is a lightweight data-interchange format that's easy for humans to read and write, and easy for machines to parse and generate.

### Using the `json` Module

Python's built-in `json` module handles JSON encoding and decoding.

#### Loading JSON from a String

```python
import json

# JSON string
json_string = '{"name": "Alice", "age": 30, "city": "New York"}'

# Parse JSON string to Python dict
data = json.loads(json_string)
print(data['name'])  # Output: Alice
print(type(data))    # Output: <class 'dict'>
```

#### Loading JSON from a File

```python
import json

# Read JSON from file
with open('data.json', 'r') as file:
    data = json.load(file)
    print(data)
```

#### Converting Python Objects to JSON

```python
import json

# Python dictionary
person = {
    'name': 'Bob',
    'age': 25,
    'hobbies': ['reading', 'coding', 'gaming'],
    'address': {
        'street': '123 Main St',
        'city': 'Boston'
    }
}

# Convert to JSON string
json_string = json.dumps(person, indent=4)
print(json_string)
```

#### Writing JSON to a File

```python
import json

data = {
    'users': [
        {'id': 1, 'name': 'Alice'},
        {'id': 2, 'name': 'Bob'}
    ]
}

# Write JSON to file
with open('output.json', 'w') as file:
    json.dump(data, file, indent=4)
```

#### Handling Complex Data Types

```python
import json
from datetime import datetime

class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

data = {
    'event': 'Meeting',
    'timestamp': datetime.now()
}

json_string = json.dumps(data, cls=DateTimeEncoder, indent=4)
print(json_string)
```

#### Parsing Nested JSON

```python
import json

json_data = '''
{
    "users": [
        {
            "id": 1,
            "name": "Alice",
            "contacts": {
                "email": "alice@example.com",
                "phone": "123-456-7890"
            }
        }
    ]
}
'''

data = json.loads(json_data)

# Access nested data
for user in data['users']:
    print(f"Name: {user['name']}")
    print(f"Email: {user['contacts']['email']}")
```

#### Error Handling

```python
import json

json_string = '{"name": "Alice", "age": 30'  # Invalid JSON (missing closing brace)

try:
    data = json.loads(json_string)
except json.JSONDecodeError as e:
    print(f"JSON parsing error: {e}")
    print(f"Error at line {e.lineno}, column {e.colno}")
```

---

## Working with External Packages

### What is pip?

**pip** is Python's package installer. It allows you to install and manage additional libraries and dependencies that aren't part of the Python standard library.

### Basic pip Commands

#### Installing a Package

```bash
# Install a specific package
pip install requests

# Install a specific version
pip install requests==2.28.0

# Install minimum version
pip install "requests>=2.25.0"

# Install from requirements file
pip install -r requirements.txt
```

#### Upgrading a Package

```bash
# Upgrade to latest version
pip install --upgrade requests

# Or use -U flag
pip install -U requests
```

#### Uninstalling a Package

```bash
pip uninstall requests

# Uninstall without confirmation
pip uninstall -y requests
```

#### Listing Installed Packages

```bash
# List all installed packages
pip list

# Show packages that need updates
pip list --outdated

# Show details about a specific package
pip show requests
```

#### Searching for Packages

```bash
# Search PyPI for packages
pip search machine-learning
```

Note: As of 2021, `pip search` has been temporarily disabled due to abuse. Use https://pypi.org/ instead.

### Understanding PyPI

**PyPI (Python Package Index)** is a repository of software for the Python programming language. It hosts thousands of packages that you can install using pip.

Website: https://pypi.org/

### Installing from Different Sources

#### From GitHub

```bash
# Install from GitHub repository
pip install git+https://github.com/username/repository.git

# Install specific branch
pip install git+https://github.com/username/repository.git@branch_name

# Install specific commit
pip install git+https://github.com/username/repository.git@commit_hash
```

#### From Local Directory

```bash
# Install package from local directory
pip install /path/to/package

# Install in editable/development mode
pip install -e /path/to/package
```

### Commonly Used Python Libraries

#### For Web Requests
- `requests` - HTTP library for humans
- `aiohttp` - Async HTTP client/server
- `httpx` - Next generation HTTP client

#### For Web Scraping
- `beautifulsoup4` - HTML/XML parser
- `scrapy` - Web scraping framework
- `selenium` - Browser automation

#### For Data Science
- `numpy` - Numerical computing
- `pandas` - Data analysis and manipulation
- `matplotlib` - Data visualization
- `scikit-learn` - Machine learning

#### For APIs
- `flask` - Micro web framework
- `fastapi` - Modern, fast web framework
- `django` - High-level web framework

---

## Virtual Environments

### Why Use Virtual Environments?

Virtual environments allow you to:
- Isolate project dependencies
- Avoid version conflicts between projects
- Maintain reproducible environments
- Test packages without affecting system Python

### Using `venv` (Built-in)

#### Creating a Virtual Environment

```bash
# Create a virtual environment named 'venv'
python -m venv venv

# Or specify a different name
python -m venv myproject_env
```

#### Activating the Virtual Environment

**On Linux/macOS:**
```bash
source venv/bin/activate
```

**On Windows:**
```bash
# Command Prompt
venv\Scripts\activate.bat

# PowerShell
venv\Scripts\Activate.ps1
```

#### Deactivating

```bash
deactivate
```

#### Installing Packages in Virtual Environment

```bash
# Activate the environment first
source venv/bin/activate

# Install packages
pip install requests numpy pandas

# Verify installation
pip list
```

### Using `virtualenv`

`virtualenv` is a more feature-rich alternative to `venv`.

#### Installation

```bash
pip install virtualenv
```

#### Usage

```bash
# Create virtual environment
virtualenv myenv

# With specific Python version
virtualenv -p python3.9 myenv

# Activate
source myenv/bin/activate  # Linux/macOS
myenv\Scripts\activate     # Windows
```

### Using `conda`

Conda is popular in data science and provides both package and environment management.

#### Creating an Environment

```bash
# Create environment with specific Python version
conda create -n myenv python=3.9

# Create with packages
conda create -n myenv python=3.9 numpy pandas

# Activate
conda activate myenv

# Deactivate
conda deactivate
```

#### Managing Packages

```bash
# Install packages
conda install requests

# List packages
conda list

# Remove package
conda remove requests
```

### Using `pipenv`

Pipenv combines pip and virtualenv into a single tool.

#### Installation

```bash
pip install pipenv
```

#### Usage

```bash
# Create environment and install packages
pipenv install requests

# Install dev dependencies
pipenv install --dev pytest

# Activate shell
pipenv shell

# Run command in environment
pipenv run python script.py
```

### Using `poetry`

Poetry is a modern dependency management tool.

#### Installation

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

#### Usage

```bash
# Initialize new project
poetry init

# Add dependency
poetry add requests

# Install dependencies
poetry install

# Run script
poetry run python script.py
```

### Virtual Environment Best Practices

1. **Always use virtual environments** for projects
2. **Don't commit** the virtual environment folder to version control
3. **Add to .gitignore**:
   ```
   venv/
   env/
   .venv/
   __pycache__/
   *.pyc
   ```
4. **Document dependencies** in requirements.txt or similar
5. **Use consistent naming** (e.g., always use 'venv' or '.venv')

---

## Dependency Management

### requirements.txt

The traditional way to manage Python dependencies.

#### Creating requirements.txt

```bash
# Generate from current environment
pip freeze > requirements.txt
```

Example `requirements.txt`:
```
requests==2.28.1
numpy==1.23.5
pandas==1.5.2
beautifulsoup4==4.11.1
```

#### Installing from requirements.txt

```bash
pip install -r requirements.txt
```

#### Best Practices for requirements.txt

1. **Pin exact versions** for reproducibility:
   ```
   requests==2.28.1
   ```

2. **Use version ranges** for flexibility:
   ```
   requests>=2.25.0,<3.0.0
   ```

3. **Separate dev dependencies**:
   ```
   # requirements-dev.txt
   pytest==7.2.0
   black==22.10.0
   flake8==6.0.0
   ```

4. **Add comments**:
   ```
   # Web scraping
   beautifulsoup4==4.11.1
   requests==2.28.1

   # Data processing
   pandas==1.5.2
   numpy==1.23.5
   ```

### setup.py and setup.cfg

For packaging and distributing your own Python projects.

#### Basic setup.py

```python
from setuptools import setup, find_packages

setup(
    name='myproject',
    version='0.1.0',
    packages=find_packages(),
    install_requires=[
        'requests>=2.25.0',
        'numpy>=1.20.0',
    ],
    extras_require={
        'dev': [
            'pytest>=6.0.0',
            'black>=22.0.0',
        ]
    },
    python_requires='>=3.7',
)
```

### pyproject.toml

Modern Python packaging standard (PEP 518).

#### Example pyproject.toml

```toml
[build-system]
requires = ["setuptools>=45", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "myproject"
version = "0.1.0"
description = "My awesome project"
requires-python = ">=3.7"
dependencies = [
    "requests>=2.25.0",
    "numpy>=1.20.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=6.0.0",
    "black>=22.0.0",
]
```

### Pipfile (pipenv)

```toml
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = ">=2.25.0"
numpy = ">=1.20.0"

[dev-packages]
pytest = "*"
black = "*"

[requires]
python_version = "3.9"
```

### poetry (pyproject.toml)

```toml
[tool.poetry]
name = "myproject"
version = "0.1.0"
description = ""

[tool.poetry.dependencies]
python = "^3.9"
requests = "^2.28.0"
numpy = "^1.23.0"

[tool.poetry.dev-dependencies]
pytest = "^7.2.0"
black = "^22.10.0"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

### Dependency Management Best Practices

1. **Use semantic versioning**:
   - `MAJOR.MINOR.PATCH`
   - `^2.0.0` means `>=2.0.0 <3.0.0`
   - `~2.0.0` means `>=2.0.0 <2.1.0`

2. **Separate production and development dependencies**

3. **Lock dependencies** for reproducibility:
   - `requirements.txt` with exact versions
   - `Pipfile.lock` (pipenv)
   - `poetry.lock` (poetry)

4. **Regularly update dependencies**:
   ```bash
   pip list --outdated
   pip install --upgrade package_name
   ```

5. **Check for security vulnerabilities**:
   ```bash
   pip install safety
   safety check
   ```

6. **Use dependency scanning** in CI/CD:
   - Dependabot (GitHub)
   - Snyk
   - PyUp

---

## Best Practices

### API Best Practices

1. **Use Environment Variables for Secrets**

   ```python
   import os
   import requests

   API_KEY = os.environ.get('API_KEY')
   headers = {'Authorization': f'Bearer {API_KEY}'}

   response = requests.get('https://api.example.com/data', headers=headers)
   ```

2. **Implement Rate Limiting**

   ```python
   import time
   import requests

   def rate_limited_request(url, calls_per_second=1):
       time.sleep(1 / calls_per_second)
       return requests.get(url)
   ```

3. **Handle Errors Gracefully**

   ```python
   import requests
   from requests.exceptions import RequestException

   def safe_api_call(url):
       try:
           response = requests.get(url, timeout=10)
           response.raise_for_status()
           return response.json()
       except requests.Timeout:
           print("Request timed out")
           return None
       except requests.HTTPError as e:
           print(f"HTTP error: {e}")
           return None
       except RequestException as e:
           print(f"Request failed: {e}")
           return None
   ```

4. **Use Retry Logic**

   ```python
   import requests
   from requests.adapters import HTTPAdapter
   from requests.packages.urllib3.util.retry import Retry

   def requests_retry_session(
       retries=3,
       backoff_factor=0.3,
       status_forcelist=(500, 502, 504),
   ):
       session = requests.Session()
       retry = Retry(
           total=retries,
           read=retries,
           connect=retries,
           backoff_factor=backoff_factor,
           status_forcelist=status_forcelist,
       )
       adapter = HTTPAdapter(max_retries=retry)
       session.mount('http://', adapter)
       session.mount('https://', adapter)
       return session

   # Usage
   response = requests_retry_session().get('https://api.example.com/data')
   ```

5. **Cache Responses**

   ```python
   import requests
   from functools import lru_cache

   @lru_cache(maxsize=100)
   def fetch_user_data(user_id):
       response = requests.get(f'https://api.example.com/users/{user_id}')
       return response.json()
   ```

### Code Organization

1. **Separate API Configuration**

   ```python
   # config.py
   import os

   class APIConfig:
       BASE_URL = 'https://api.example.com'
       API_KEY = os.environ.get('API_KEY')
       TIMEOUT = 10
       MAX_RETRIES = 3
   ```

2. **Create API Client Classes**

   ```python
   # api_client.py
   import requests
   from config import APIConfig

   class APIClient:
       def __init__(self):
           self.base_url = APIConfig.BASE_URL
           self.headers = {
               'Authorization': f'Bearer {APIConfig.API_KEY}',
               'Content-Type': 'application/json'
           }

       def get(self, endpoint):
           url = f'{self.base_url}/{endpoint}'
           response = requests.get(
               url,
               headers=self.headers,
               timeout=APIConfig.TIMEOUT
           )
           response.raise_for_status()
           return response.json()

       def post(self, endpoint, data):
           url = f'{self.base_url}/{endpoint}'
           response = requests.post(
               url,
               json=data,
               headers=self.headers,
               timeout=APIConfig.TIMEOUT
           )
           response.raise_for_status()
           return response.json()
   ```

3. **Use Type Hints**

   ```python
   from typing import Dict, List, Optional
   import requests

   def fetch_users(limit: int = 10) -> List[Dict[str, any]]:
       response = requests.get(
           'https://api.example.com/users',
           params={'limit': limit}
       )
       return response.json()

   def get_user_by_id(user_id: int) -> Optional[Dict[str, any]]:
       response = requests.get(f'https://api.example.com/users/{user_id}')
       if response.status_code == 404:
           return None
       return response.json()
   ```

### Security Best Practices

1. **Never commit API keys or secrets**
2. **Use environment variables** (.env file with python-dotenv)
3. **Validate and sanitize** user input
4. **Use HTTPS** for all API calls
5. **Implement authentication** properly
6. **Keep dependencies updated** for security patches

### Testing API Code

```python
import unittest
from unittest.mock import patch, Mock
import requests

class TestAPIClient(unittest.TestCase):
    @patch('requests.get')
    def test_successful_request(self, mock_get):
        # Mock the response
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = {'id': 1, 'name': 'Alice'}
        mock_get.return_value = mock_response

        # Test your function
        response = requests.get('https://api.example.com/users/1')
        data = response.json()

        self.assertEqual(data['name'], 'Alice')
        mock_get.assert_called_once_with('https://api.example.com/users/1')

if __name__ == '__main__':
    unittest.main()
```

---

## Practical Examples

### Example 1: Weather API Client

```python
import requests
import os
from typing import Dict, Optional

class WeatherClient:
    """A client for fetching weather data from OpenWeatherMap API"""

    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = 'https://api.openweathermap.org/data/2.5'

    def get_current_weather(self, city: str) -> Optional[Dict]:
        """Get current weather for a city"""
        endpoint = f'{self.base_url}/weather'
        params = {
            'q': city,
            'appid': self.api_key,
            'units': 'metric'
        }

        try:
            response = requests.get(endpoint, params=params, timeout=10)
            response.raise_for_status()
            return response.json()
        except requests.RequestException as e:
            print(f"Error fetching weather: {e}")
            return None

    def format_weather_data(self, data: Dict) -> str:
        """Format weather data into readable string"""
        if not data:
            return "No weather data available"

        city = data['name']
        temp = data['main']['temp']
        description = data['weather'][0]['description']
        humidity = data['main']['humidity']

        return f"""
        Weather in {city}:
        Temperature: {temp}°C
        Condition: {description}
        Humidity: {humidity}%
        """

# Usage
if __name__ == '__main__':
    API_KEY = os.environ.get('OPENWEATHER_API_KEY')
    client = WeatherClient(API_KEY)

    weather = client.get_current_weather('London')
    print(client.format_weather_data(weather))
```

### Example 2: GitHub API Integration

```python
import requests
from typing import List, Dict

class GitHubClient:
    """Client for interacting with GitHub API"""

    def __init__(self, token: str = None):
        self.base_url = 'https://api.github.com'
        self.headers = {
            'Accept': 'application/vnd.github.v3+json'
        }
        if token:
            self.headers['Authorization'] = f'token {token}'

    def get_user_repos(self, username: str) -> List[Dict]:
        """Get all public repositories for a user"""
        url = f'{self.base_url}/users/{username}/repos'
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()

    def get_repo_languages(self, owner: str, repo: str) -> Dict:
        """Get programming languages used in a repository"""
        url = f'{self.base_url}/repos/{owner}/{repo}/languages'
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()

    def search_repositories(self, query: str, sort: str = 'stars') -> List[Dict]:
        """Search for repositories"""
        url = f'{self.base_url}/search/repositories'
        params = {
            'q': query,
            'sort': sort,
            'order': 'desc'
        }
        response = requests.get(url, headers=self.headers, params=params)
        response.raise_for_status()
        return response.json()['items']

# Usage
if __name__ == '__main__':
    client = GitHubClient()

    # Get repos for a user
    repos = client.get_user_repos('torvalds')
    print(f"Found {len(repos)} repositories")

    # Search for Python repos
    results = client.search_repositories('machine learning language:python')
    for repo in results[:5]:
        print(f"{repo['name']}: {repo['stargazers_count']} stars")
```

### Example 3: REST API with Error Handling

```python
import requests
import logging
from typing import Optional, Dict, Any
from requests.exceptions import HTTPError, ConnectionError, Timeout, RequestException

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class APIClient:
    """Robust API client with comprehensive error handling"""

    def __init__(self, base_url: str, api_key: str = None):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()

        if api_key:
            self.session.headers.update({
                'Authorization': f'Bearer {api_key}'
            })

        self.session.headers.update({
            'Content-Type': 'application/json',
            'User-Agent': 'Python API Client/1.0'
        })

    def _make_request(
        self,
        method: str,
        endpoint: str,
        **kwargs
    ) -> Optional[Dict[str, Any]]:
        """Make HTTP request with error handling"""
        url = f'{self.base_url}/{endpoint.lstrip("/")}'

        try:
            response = self.session.request(
                method=method,
                url=url,
                timeout=kwargs.get('timeout', 30),
                **{k: v for k, v in kwargs.items() if k != 'timeout'}
            )

            response.raise_for_status()

            # Log successful request
            logger.info(f"{method} {url} - Status: {response.status_code}")

            # Return JSON if response has content
            if response.content:
                return response.json()
            return {'status': 'success'}

        except HTTPError as e:
            logger.error(f"HTTP Error: {e}")
            if e.response.status_code == 401:
                logger.error("Authentication failed")
            elif e.response.status_code == 404:
                logger.error("Resource not found")
            elif e.response.status_code == 429:
                logger.error("Rate limit exceeded")
            return None

        except ConnectionError as e:
            logger.error(f"Connection Error: {e}")
            return None

        except Timeout as e:
            logger.error(f"Request Timeout: {e}")
            return None

        except RequestException as e:
            logger.error(f"Request failed: {e}")
            return None

    def get(self, endpoint: str, params: Dict = None) -> Optional[Dict]:
        """Make GET request"""
        return self._make_request('GET', endpoint, params=params)

    def post(self, endpoint: str, data: Dict = None) -> Optional[Dict]:
        """Make POST request"""
        return self._make_request('POST', endpoint, json=data)

    def put(self, endpoint: str, data: Dict = None) -> Optional[Dict]:
        """Make PUT request"""
        return self._make_request('PUT', endpoint, json=data)

    def delete(self, endpoint: str) -> Optional[Dict]:
        """Make DELETE request"""
        return self._make_request('DELETE', endpoint)

    def close(self):
        """Close the session"""
        self.session.close()

# Usage
if __name__ == '__main__':
    client = APIClient('https://api.example.com')

    # GET request
    data = client.get('/users/1')
    if data:
        print(f"User: {data}")

    # POST request
    new_user = {
        'name': 'John Doe',
        'email': 'john@example.com'
    }
    result = client.post('/users', data=new_user)

    # Clean up
    client.close()
```

### Example 4: Working with Environment Variables

```python
# .env file
"""
API_KEY=your_api_key_here
API_SECRET=your_secret_here
DATABASE_URL=postgresql://localhost/mydb
DEBUG=True
"""

# Python code
import os
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

class Config:
    """Application configuration from environment variables"""

    API_KEY = os.getenv('API_KEY')
    API_SECRET = os.getenv('API_SECRET')
    DATABASE_URL = os.getenv('DATABASE_URL')
    DEBUG = os.getenv('DEBUG', 'False').lower() == 'true'

    @classmethod
    def validate(cls):
        """Validate required configuration"""
        required = ['API_KEY', 'DATABASE_URL']
        missing = [key for key in required if not getattr(cls, key)]

        if missing:
            raise ValueError(f"Missing required config: {', '.join(missing)}")

# Usage
if __name__ == '__main__':
    Config.validate()
    print(f"API Key loaded: {Config.API_KEY[:5]}...")
    print(f"Debug mode: {Config.DEBUG}")
```

### Example 5: Async API Calls for Performance

```python
import asyncio
import aiohttp
from typing import List, Dict
import time

class AsyncAPIClient:
    """Asynchronous API client for concurrent requests"""

    def __init__(self, base_url: str):
        self.base_url = base_url

    async def fetch(self, session: aiohttp.ClientSession, url: str) -> Dict:
        """Fetch a single URL"""
        async with session.get(url) as response:
            return await response.json()

    async def fetch_multiple(self, endpoints: List[str]) -> List[Dict]:
        """Fetch multiple endpoints concurrently"""
        async with aiohttp.ClientSession() as session:
            tasks = []
            for endpoint in endpoints:
                url = f"{self.base_url}/{endpoint}"
                tasks.append(self.fetch(session, url))

            return await asyncio.gather(*tasks)

# Usage example
async def main():
    client = AsyncAPIClient('https://jsonplaceholder.typicode.com')

    # Fetch multiple users concurrently
    endpoints = [f'users/{i}' for i in range(1, 11)]

    start_time = time.time()
    results = await client.fetch_multiple(endpoints)
    end_time = time.time()

    print(f"Fetched {len(results)} users in {end_time - start_time:.2f} seconds")

    for user in results:
        print(f"- {user['name']}")

# Run the async function
if __name__ == '__main__':
    asyncio.run(main())
```

### Example 6: Complete Project Structure

```
my_project/
│
├── .env                      # Environment variables (not in git)
├── .gitignore               # Git ignore file
├── requirements.txt         # Production dependencies
├── requirements-dev.txt     # Development dependencies
├── README.md               # Project documentation
│
├── config/
│   ├── __init__.py
│   └── settings.py         # Configuration management
│
├── src/
│   ├── __init__.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── client.py       # API client
│   │   └── endpoints.py    # API endpoints
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py         # Data models
│   │
│   └── utils/
│       ├── __init__.py
│       ├── logger.py       # Logging utilities
│       └── validators.py   # Input validation
│
├── tests/
│   ├── __init__.py
│   ├── test_api.py
│   └── test_models.py
│
└── main.py                 # Entry point
```

---

## Summary

This guide covered:

1. **APIs**: Understanding REST APIs and HTTP methods
2. **HTTP Requests**: Using `requests`, `urllib`, and `aiohttp` libraries
3. **JSON**: Parsing and generating JSON data
4. **External Packages**: Installing and managing packages with pip
5. **Virtual Environments**: Creating isolated Python environments with venv, virtualenv, conda, pipenv, and poetry
6. **Dependency Management**: Using requirements.txt, setup.py, and pyproject.toml
7. **Best Practices**: Error handling, security, code organization, and testing
8. **Practical Examples**: Real-world implementations of API clients

### Key Takeaways

- Always use virtual environments for projects
- Manage dependencies explicitly with requirements.txt or similar
- Handle API errors gracefully with try-except blocks
- Never commit API keys or secrets to version control
- Use environment variables for configuration
- Implement proper logging and error handling
- Write testable code with clear separation of concerns
- Keep dependencies updated for security

### Next Steps

1. Practice making API calls with public APIs (GitHub, OpenWeather, etc.)
2. Build a small project that integrates with an external API
3. Learn about authentication methods (OAuth, JWT, API keys)
4. Explore asynchronous programming for better performance
5. Study API design if you plan to build your own APIs

### Recommended Resources

- **Requests Documentation**: https://requests.readthedocs.io/
- **Python Packaging Guide**: https://packaging.python.org/
- **Real Python Tutorials**: https://realpython.com/
- **Public APIs for Practice**: https://github.com/public-apis/public-apis
- **Python Virtual Environments**: https://docs.python.org/3/tutorial/venv.html

---

*Last Updated: January 2026*
