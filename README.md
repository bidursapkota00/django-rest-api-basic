# Django Rest API - Advanced

![Bidur Sapkota](https://www.bidursapkota.com.np/images/gravatar.webp "Bidur Sapkota - Developer")&nbsp;[Bidur Sapkota](https://www.bidursapkota.com.np/)

![Django Rest API Advanced Guide by Bidur Sapkota](/test.webp "Django Rest API Advanced Guide – Blog by Bidur Sapkota")

## Table of Contents

1. [Introduction and Installation](#Introduction and Installation)

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
docker-compose --version
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

**Docker Hub**

- Go to `hub.docker.com`
- Create account
- Go to account settings
- Select Security tab, then click New Access Token

**GitHub**

- Go to repo's settings
- Select Secrets tab, then click New repository secret
- Add secret: DOCKERHUB_USER. Set username of DockerHub
- Add secret: DOCKERHUB_TOKEN. Set token generated in above step

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

**Using Docker Compose**

- Run all commands through Docker Compose

```bash
  docker-compose run --rm app sh -c "python manage.py collectstatic"
```

- `docker-compose` runs a Docker Compose command
- `run` will start a specific container defined in config
- `--rm` removes the container
- `app` is the name of the service
- `sh -c` passess in a shell command
- Command to run inside container

**Create requirements.txt**

```text
Django>=3.2.4,<3.3
djangorestframework>=3.12.4,<3.13
```
