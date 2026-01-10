# Django Rest API - Advanced

![Bidur Sapkota](https://www.bidursapkota.com.np/images/gravatar.webp "Bidur Sapkota - Developer")&nbsp;[Bidur Sapkota](https://www.bidursapkota.com.np/)

![Django Rest API Advanced Guide by Bidur Sapkota](/test.webp "Django Rest API Advanced Guide – Blog by Bidur Sapkota")

## Table of Contents

1. [Introduction and Installation](#introduction-and-installation)
2. [First GitHub Actions](#first-github-actions)
3. [First Tests](#first-tests)

## Introduction and Installation

**What are we building?**

- Backend API (Recipe API)
- With Python, Django, Django REST Framework (one container)
- PostgreSQL for database (one container)
- Also browsable documentation with Swagger / OpenAPI
- Will use Docker to create development container environment
- Browseable Admin Interface (Django Admin)
- Linting and Automated Testing

**Apps**

- app/ - Django project
- app/core/ - Code shared between multiple apps
- app/user/ - User related code like user authentication
- app/recipe/ - Recipe related code like managing recipes, tags, ingredients

**Unit / Integration Tests**

- Code with tests code
  - sets up conditions / inputs
  - Runs a piece of code
  - Checks outputs with 'assertions'
- Benefits
  - Ensures code runs as expected
  - catches bugs
  - Improves reliability
  - Provides confidence

**Test Driven Development(TDD) Process**

![Test Driven Development](/images/tdd.webp)

**Why TDD?**

- Better understanding of code
- Make changes with confidence
- Reduces bugs

**Applications to install**

- VSCode
- Docker
- Git

**Test of installation**

```bash
docker --version
docker compose --version
git --version
```

**Why Docker**

- Consistent dev and prod environment
- Easier collaboration
- Capture all dependencies as code
  - Python requirements
  - Operating system dependencies
- Easier cleanup
- Save a lot of time
- **Drawbacks**
  - VSCode unable to access interpreter
  - More difficult to use integrated features
  - Suggest using Terminal

**How use Docker**

- Define Dockerfile
- Create Docker Compose configuration
- Run all commands via Docker Compose

**Docker, Docker Hub and GitHub Actions**

- Docker Hub introduced rate limit:
  - 100 pulls / 6hr for unauthenticated users
  - 200 pull/6hr for authenticated users
- GitHub Actions is a shared service
  - 100 pulls / 6hr applied for all users
- Authenticate with Docker Hub
  - Create account
  - Setup credentials
  - Login before running job
  - Get 200 pulls / 6hr for free!

**Create New Repo on GitHub**

- Add .gitignore
- Select .gitignore template: Python
- Clone repo locally
- Open VSCode on cloned folder

**Create requirements.txt**

```text
Django>=3.2.4,<3.3
djangorestframework>=3.12.4,<3.13
```

**Configure Docker**

- Create a Dockerfile
- Create Image
  - Choose a base image (python)
  - Install dependencies
  - Setup users

```Dockerfile
FROM python:3.9-alpine3.13
LABEL maintainer="bidursapkota.com.np"

ENV PYTHONUNBUFFERED 1

COPY ./requirements.txt /tmp/requirements.txt
COPY ./app /app
WORKDIR /app
EXPOSE 8000

RUN python -m venv /py && \
    /py/bin/pip install --upgrade pip && \
    /py/bin/pip install -r /tmp/requirements.txt && \
    rm -rf /tmp && \
    adduser \
        --disabled-password \
        --no-create-home \
        django-user

ENV PATH="/py/bin:$PATH"

USER django-user
```

**Create .dockerignore file**

```dockerignore
# Git
.git
.gitignore

# Docker
.docker

# Python
app/__pycache__/
app/*/__pycache__/
app/**/*/__pycache__/
app/**/**/__pycache__/
.env/
.venv/
venv/
```

**Build Image**

- Make sure to be in project root folder.
- <font color='red'>Since: COPY ./app /app.<br>Create empty app folder</font>

```bash
docker build .
```

**Docker Compose**

- How our Docker image(s) should be used
- Define our 'services'
  - Name (eg: app)
  - Port mappings
  - Volume mappings

**Create docker-compose.yml**

```yml
version: "3.9"

services:
  app:
    build:
      context: .
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000"
```

**Build Image**

- does same as docker build .

```bash
docker compose build
```

**Using Docker Compose**

- Run all commands through Docker Compose

```bash
  docker compose run --rm app sh -c "python manage.py collectstatic"
```

- `docker compose` runs a Docker Compose command
- `run` will start a specific container defined in config
- `--rm` removes the container
- `app` is the name of the service
- `sh -c` passess in a shell command
- Command to run inside container

**Linting**

- Tools to check code formatting
- Highlights errors, typos, formatting issues
- **Install flake8 package**
- Run it through Docker Compose

```bash
docker compose run --rm app sh -c "flake8"
```

**Testing**

- Django test suite
- Setup tests per Django app
- Run tests through Docker Compose

```bash
docker compose run --rm app sh -c "python manage.py test"
```

**Create file requirements.dev.txt**

```text
flake8>=3.9.2,<3.10
```

**Update Dockerfile**

```dockerfile
FROM python:3.9-alpine3.13
LABEL maintainer="bidursapkota.com.np"

ENV PYTHONUNBUFFERED 1

COPY ./requirements.txt /tmp/requirements.txt
COPY ./requirements.dev.txt /tmp/requirements.dev.txt
COPY ./app /app
WORKDIR /app
EXPOSE 8000

ARG DEV=false
RUN python -m venv /py && \
    /py/bin/pip install --upgrade pip && \
    /py/bin/pip install -r /tmp/requirements.txt && \
    if [ $DEV = "true" ]; \
        then /py/bin/pip install -r /tmp/requirements.dev.txt ; \
    fi && \
    rm -rf /tmp && \
    adduser \
        --disabled-password \
        --no-create-home \
        django-user

ENV PATH="/py/bin:$PATH"

USER django-user
```

**Update docker-compose.yml**

```yml
version: "3.9"

services:
  app:
    build:
      context: .
      args:
        - DEV=true
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000"
```

**Build Image**

```bash
docker compose build
```

**Create file app/.flake8**

```flake8
[flake8]
exclude =
  migrations,
  __pycache__,
  manage.py,
  settings.py
```

```bash
docker compose run --rm app sh -c "flake8"
```

**Create Django Project**

```bash
docker compose run --rm app sh -c "django-admin startproject app ."
```

**Run Django Project**

```bash
docker compose up
```

Visit `127.0.0.1:8000`

---

---

---

## First GitHub Actions

**GitHub Actions**

- Automation tool
- Similar to Travis-CI, GitLab CI/CD, Jenkins
- Run jobs when code changes
- Automate tasks

**Common uses**

- Deployment
- Code linting
- Unit tests

**How it works**

- **Trigger:** Push to GitHub
- **Job:** Run unit tests
- **Result:** Success/fail

**Pricing**

- Charged per minutes
- 2,000 free minutes
- Various plans available

**Docker Hub**

- Needed to pull base images
- Authenticate with Docker Hub, then 200 pulls per 6h is available
- Go to `hub.docker.com`
- Create account
- Go to account settings
- Select Security tab, then click New Access Token

**On GitHub**

- Go to repo's settings
- Select Secrets tab, then click New repository secret
- Add secret: DOCKERHUB_USER. Set username of DockerHub
- Add secret: DOCKERHUB_TOKEN. Set token generated in above step

**How we’ll configure GitHub Actions**

- Create a config file at `.github/workflows/checks.yml`
- Set trigger
- Configure Docker Hub auth
- Add steps for running testing and linting

**How to authenticate with Docker Hub?**

- Register account on [https://hub.docker.com/](https://hub.docker.com/)
- Use `docker login` during our job
- Add secrets to GitHub project
- Secrets are encrypted
- Decrypted when needed in actions

**Create .github/workflows/checks.yml**

```yml
---
name: Checks

on: push

jobs:
  test-lint:
    name: Test and Lint
    runs-on: ubuntu-latest

    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Checkout
        uses: actions/checkout@v2
      - name: Test
        run: docker compose run --rm app sh -c "python manage.py test"
      - name: Lint
        run: docker compose run --rm app sh -c "flake8"
```

---

---

---

## First Tests

**Django test framework**

- Based on the `unittest` library
- Django adds features:

  - Test client - dummy web browser
  - Simulate authentication
  - Temporary database

- Django REST Framework adds features:

  - API test client

**Where do you put tests?**

- Placeholder `tests.py` added to each app
- Or, create `tests/` subdirectory to split tests up
- Keep in mind:
  - Only use `tests.py` or `tests/` directory (not both)
  - Test modules must start with `test_`
  - Test directories must contain `__init__.py`

**Test Database**

- Test code that uses the DB
- Specific database for tests
- Runs test then Clears data (loops back to Runs test)
- Happens for _every_ test (by default)

---

**Test classes**

- **`SimpleTestCase`**

  - No database integration
  - Useful if no database is required for your test
  - Save time executing tests

- **`TestCase`**

  - Database integration
  - Useful for testing code that uses the database

**Writing tests**

- Import test class

  - `SimpleTestCase` - No database
  - `TestCase` - Database

- Import objects to test
- Define test class
- Add test method
- Setup inputs
- Execute code to be tested
- Check output

**Example**

**Create file app/app/calc.py**

```py
"""
Calculator functions
"""


def add(x, y):
    """Add x and y and return result."""
    return x + y


def subtract(x, y):
    """Subtract x from y and return result."""
    return y - x
```

**Create file app/app/tests.py**

```py
"""
Sample tests
"""
from django.test import SimpleTestCase

from app import calc


class CalcTests(SimpleTestCase):
    """Test the calc module."""

    def test_add_numbers(self):
        """Test adding numbers together."""
        res = calc.add(5, 6)

        self.assertEqual(res, 11)

    def test_subtract_numbers(self):
        """Test subtracting numbers."""
        res = calc.subtract(10, 15)

        self.assertEqual(res, 5)
```

**Run Tests**

```bash
docker compose run --rm app sh -c "python manage.py test"
```

---

**What is Mocking?**

- Override or change behaviour of dependencies
- Avoid unintended side effects
- Isolate code being tested

---

**Why use mocking?**

- Avoid relying on external services

  - Can't guarantee they will be available
  - Makes tests unpredictable and inconsistent

- Avoid unintended consequences

  - Accidentally sending emails
  - Overloading external services

**How to mock code?**

- **Use `unittest.mock**`
  - `MagicMock` / `Mock` - Replace real objects
  - `patch` - Overrides code for tests

---

**Testing APIs**

- Make actual requests
- Check result

---

**Django REST Framework APIClient**

- Based on the Django’s `TestClient`
- Make requests
- Check result
- Override authentication

**Code Implementation Example**

```python
from django.test import SimpleTestCase
from rest_framework.test import APIClient

class TestViews(SimpleTestCase):

    def test_get_greetings(self):
        """ Test getting greetings. """
        client = APIClient()
        res = client.get('/greetings/')

        self.assertEqual(res.status_code, 200)
        self.assertEqual(
            res.data,
            ["Hello!"],
        )
```

**Possible reasons for tests not running**

- Missing `__init__.py` in `tests/` dir
- Indentation of test cases
- Missing `test` prefix for method
- Both `tests/` directory and `tests.py` exist (ImportError)

---

---

---

**Probable CI/CD**

```yml
name: Django CI/CD

on:
  push:
    branches:
      - main
  workflow_dispatch: # allows manual trigger

jobs:
  test-lint:
    name: Test and Lint
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: python manage.py test

      - name: Run flake8
        run: flake8

  build-push:
    name: Build & Push Docker
    needs: test-lint
    runs-on: ubuntu-latest
    if: success() # only run if test-lint passes

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker Image
        run: docker build -t ${{ secrets.DOCKERHUB_USER }}/myapp:latest .

      - name: Push Docker Image
        run: docker push ${{ secrets.DOCKERHUB_USER }}/myapp:latest

  deploy:
    name: Deploy to Render via SSH
    needs: build-push
    runs-on: ubuntu-latest
    if: github.event_name == 'workflow_dispatch' # only manual trigger

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.RENDER_HOST }}
          username: ${{ secrets.RENDER_USER }}
          key: ${{ secrets.RENDER_SSH_KEY }}
          script: |
            docker pull ${{ secrets.DOCKERHUB_USER }}/myapp:latest
            docker compose down
            docker compose up -d
```

**Required GitHub Secrets**

```text
DOCKERHUB_USER
DOCKERHUB_TOKEN
RENDER_HOST         # IP / hostname of your Render instance
RENDER_USER         # SSH username
RENDER_SSH_KEY      # private SSH key for Render instance
```
