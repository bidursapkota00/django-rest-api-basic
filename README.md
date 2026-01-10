# Django REST API - Complete Step-by-Step Course

A comprehensive Recipe API built with Django REST Framework, Docker, PostgreSQL, and GitHub Actions CI/CD.

## 📋 Table of Contents

1. [Project Overview](#-project-overview)
2. [Prerequisites](#-prerequisites)
3. [Part 1: Project Setup & Docker Configuration](#part-1-project-setup--docker-configuration)
4. [Part 2: GitHub Actions CI/CD](#part-2-github-actions-cicd)
5. [Part 3: Core App - Custom User Model](#part-3-core-app---custom-user-model)
6. [Part 4: Wait for DB Command](#part-4-wait-for-db-command)
7. [Part 5: User API](#part-5-user-api)
8. [Part 6: Recipe API](#part-6-recipe-api)
9. [Part 7: Tags & Ingredients API](#part-7-tags--ingredients-api)
10. [Part 8: Image Upload](#part-8-image-upload)
11. [Part 9: API Documentation](#part-9-api-documentation)

---

## 🎯 Project Overview

### Features

- Custom User Model with email authentication
- Token-based authentication
- Recipe CRUD operations with filtering
- Tags and Ingredients management
- Image upload for recipes
- Swagger/OpenAPI documentation
- Docker & Docker Compose setup
- PostgreSQL database
- GitHub Actions CI/CD
- Flake8 linting

### Project Structure

```
django-rest-api-basic/
├── .github/workflows/checks.yml    # CI/CD pipeline
├── app/
│   ├── app/                        # Django project settings
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── core/                       # Core app (User model, admin)
│   │   ├── management/commands/    # Custom commands
│   │   ├── tests/
│   │   ├── models.py
│   │   └── admin.py
│   ├── user/                       # User API
│   │   ├── tests/
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── recipe/                     # Recipe API
│   │   ├── tests/
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── .flake8
│   └── manage.py
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── requirements.dev.txt
```

---

## 🔧 Prerequisites

- Docker Desktop installed
- Docker Hub account (for CI/CD)
- Git installed
- Code editor (VS Code recommended)

---

# Part 1: Project Setup & Docker Configuration

## Step 1.1: Create Project Directory

```bash
mkdir django-rest-api-basic
cd django-rest-api-basic
```

## Step 1.2: Create requirements.txt

```bash
echo "Django>=3.2.4,<3.3
djangorestframework>=3.12.4,<3.13
psycopg2>=2.8.6,<2.9
drf-spectacular>=0.15.1,<0.16
Pillow>=8.2.0,<8.3.0" > requirements.txt
```

## Step 1.3: Create requirements.dev.txt

```bash
echo "flake8>=3.9.2,<3.10" > requirements.dev.txt
```

## Step 1.4: Create Dockerfile

Create `Dockerfile` with the following content:

```dockerfile
FROM python:3.9-alpine3.13
LABEL maintainer="your-name.com"

ENV PYTHONUNBUFFERED=1

COPY ./requirements.txt /tmp/requirements.txt
COPY ./requirements.dev.txt /tmp/requirements.dev.txt
COPY ./app /app
WORKDIR /app
EXPOSE 8000

ARG DEV=false
RUN python -m venv /py && \
    /py/bin/pip install --upgrade pip && \
    apk add --update --no-cache postgresql-client jpeg-dev && \
    apk add --update --no-cache --virtual .tmp-build-deps \
        build-base postgresql-dev musl-dev zlib zlib-dev && \
    /py/bin/pip install -r /tmp/requirements.txt && \
    if [ $DEV = "true" ]; \
        then /py/bin/pip install -r /tmp/requirements.dev.txt ; \
    fi && \
    rm -rf /tmp && \
    apk del .tmp-build-deps && \
    adduser \
    --disabled-password \
    --no-create-home \
    django-user && \
    mkdir -p /vol/web/media && \
    mkdir -p /vol/web/static && \
    chown -R django-user:django-user /vol && \
    chmod -R 755 /vol

ENV PATH="/py/bin:$PATH"

USER django-user
```

## Step 1.5: Create docker-compose.yml

```yaml
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
      - dev-static-data:/vol/web
    command: >
      sh -c "python manage.py wait_for_db &&
             python manage.py migrate &&
             python manage.py runserver 0.0.0.0:8000"
    environment:
      - DB_HOST=db
      - DB_NAME=devdb
      - DB_USER=devuser
      - DB_PASS=changeme
    depends_on:
      - db

  db:
    image: postgres:13-alpine
    volumes:
      - dev-db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=devdb
      - POSTGRES_USER=devuser
      - POSTGRES_PASSWORD=changeme

volumes:
  dev-db-data:
  dev-static-data:
```

## Step 1.6: Create .dockerignore

```bash
echo "# Git
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
venv/" > .dockerignore
```

## Step 1.7: Create Django Project

```bash
# Create app directory
mkdir app

# Build Docker image
docker compose build

# Start Django project
docker compose run --rm app sh -c "django-admin startproject app ."
```

## Step 1.8: Create .flake8 Configuration

```bash
echo "[flake8]
exclude =
    migrations,
    __pycache__,
    manage.py,
    settings.py" > app/.flake8
```

## Step 1.9: Update settings.py

Update `app/app/settings.py` with these changes:

```python
# Add at top
import os

# Update DATABASES
DATABASES = {
    'default': {
        "ENGINE": "django.db.backends.postgresql",
        "HOST": os.environ.get("DB_HOST"),
        "NAME": os.environ.get("DB_NAME"),
        "USER": os.environ.get("DB_USER"),
        "PASSWORD": os.environ.get("DB_PASS"),
    }
}

# Add static/media settings at bottom
STATIC_URL = '/static/static/'
MEDIA_URL = '/static/media/'
MEDIA_ROOT = '/vol/web/media'
STATIC_ROOT = '/vol/web/static'
```

### ✅ Verification - Build & Lint

```bash
# Build the image
docker compose build

# Run flake8 linting
docker compose run --rm app sh -c "flake8"
```

---

# Part 2: GitHub Actions CI/CD

## Step 2.1: Create GitHub Workflow

```bash
mkdir -p .github/workflows
```

Create `.github/workflows/checks.yml`:

```yaml
---
name: Checks

on: push

jobs:
  test-lint:
    name: Test and Lint
    runs-on: ubuntu-20.04

    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Checkout
        uses: actions/checkout@v2
      - name: Test
        run: docker compose run --rm app sh -c "python manage.py wait_for_db && python manage.py test"
      - name: Lint
        run: docker compose run --rm app sh -c "flake8"
```

## Step 2.2: Configure GitHub Secrets

1. Go to your GitHub repository → Settings → Secrets → Actions
2. Add these secrets:
   - `DOCKERHUB_USER`: Your Docker Hub username
   - `DOCKERHUB_TOKEN`: Your Docker Hub access token

---

# Part 3: Core App - Custom User Model

## Step 3.1: Create Core App

```bash
docker compose run --rm app sh -c "python manage.py startapp core"
```

## Step 3.2: Remove Unused Files

Delete these files from `app/core/`:

- `views.py`
- `tests.py` (we'll create a tests folder)

```bash
# Create tests directory
mkdir app/core/tests
echo "" > app/core/tests/__init__.py
```

## Step 3.3: Update settings.py - Add Core App

Add to `INSTALLED_APPS` in `app/app/settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core',  # Add this
]

# Add at bottom
AUTH_USER_MODEL = 'core.User'
```

## Step 3.4: Create User Model Tests

Create `app/core/tests/test_models.py`:

```python
"""
Tests for models.
"""
from unittest.mock import patch
from decimal import Decimal

from django.test import TestCase
from django.contrib.auth import get_user_model

from core import models


def create_user(email='user@example.com', password='testpass123'):
    """Create and return a new user."""
    return get_user_model().objects.create_user(email, password)


class ModelTests(TestCase):
    """Test models."""

    def test_create_user_with_email_successful(self):
        """Test creating a user with an email is successful."""
        email = 'test@example.com'
        password = 'testpass123'
        user = get_user_model().objects.create_user(
            email=email,
            password=password,
        )

        self.assertEqual(user.email, email)
        self.assertTrue(user.check_password(password))

    def test_new_user_email_normalized(self):
        """Test email is normalized for new users."""
        sample_emails = [
            ['test1@EXAMPLE.com', 'test1@example.com'],
            ['Test2@Example.com', 'Test2@example.com'],
            ['TEST3@EXAMPLE.COM', 'TEST3@example.com'],
            ['test4@example.COM', 'test4@example.com'],
        ]
        for email, expected in sample_emails:
            user = get_user_model().objects.create_user(email, 'sample123')
            self.assertEqual(user.email, expected)

    def test_new_user_without_email_raises_error(self):
        """Test that creating a user without an email raises a ValueError."""
        with self.assertRaises(ValueError):
            get_user_model().objects.create_user('', 'test123')

    def test_create_superuser(self):
        """Test creating a superuser."""
        user = get_user_model().objects.create_superuser(
            'test@example.com',
            'test123',
        )
        self.assertTrue(user.is_superuser)
        self.assertTrue(user.is_staff)
```

## Step 3.5: Create User Model

Update `app/core/models.py`:

```python
from django.contrib.auth.models import (
    AbstractBaseUser,
    BaseUserManager,
    PermissionsMixin,
)
from django.db import models


class UserManager(BaseUserManager):
    """Manager for users."""

    def create_user(self, email, password=None, **extra_fields):
        """Create, save and return a new user."""
        if not email:
            raise ValueError('User must have an email address.')
        user = self.model(email=self.normalize_email(email), **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password):
        """Create and return a new superuser."""
        user = self.create_user(email, password)
        user.is_staff = True
        user.is_superuser = True
        user.save(using=self._db)
        return user


class User(AbstractBaseUser, PermissionsMixin):
    """User in the system."""
    email = models.EmailField(max_length=255, unique=True)
    name = models.CharField(max_length=255)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    objects = UserManager()

    USERNAME_FIELD = 'email'
```

## Step 3.6: Create and Run Migrations

```bash
docker compose run --rm app sh -c "python manage.py makemigrations"
```

### ✅ Verification - Run Tests

```bash
docker compose run --rm app sh -c "python manage.py test"
docker compose run --rm app sh -c "flake8"
```

## Step 3.7: Setup Django Admin

Create `app/core/tests/test_admin.py`:

```python
"""Tests for the Django admin modifications."""

from django.test import TestCase
from django.contrib.auth import get_user_model
from django.urls import reverse
from django.test import Client


class AdminSiteTests(TestCase):
    """Tests for Django admin."""

    def setUp(self):
        """Create user and client."""
        self.client = Client()
        self.admin_user = get_user_model().objects.create_superuser(
            email='admin@example.com',
            password='testpass123',
        )
        self.client.force_login(self.admin_user)
        self.user = get_user_model().objects.create_user(
            email='user@example.com',
            password='testpass123',
            name='Test User'
        )

    def test_users_list(self):
        """Test that users are listed on page."""
        url = reverse('admin:core_user_changelist')
        res = self.client.get(url)
        self.assertContains(res, self.user.name)
        self.assertContains(res, self.user.email)

    def test_edit_user_page(self):
        """Test the edit user page works."""
        url = reverse('admin:core_user_change', args=[self.user.id])
        res = self.client.get(url)
        self.assertEqual(res.status_code, 200)

    def test_create_user_page(self):
        """Test the create user page works."""
        url = reverse('admin:core_user_add')
        res = self.client.get(url)
        self.assertEqual(res.status_code, 200)
```

Update `app/core/admin.py`:

```python
"""Customize django admin page"""

from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.utils.translation import gettext_lazy as _
from core import models


class UserAdmin(BaseUserAdmin):
    """Define the admin pages for users."""
    ordering = ['id']
    list_display = ['email', 'name']
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        (_('Permissions'), {
            'fields': (
                'is_active',
                'is_staff',
                'is_superuser',
            ),
        }),
        (_('Important dates'), {'fields': ('last_login',)}),
    )
    readonly_fields = ['last_login']
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': (
                'email',
                'password1',
                'password2',
                'name',
                'is_active',
                'is_staff',
                'is_superuser',
            ),
        }),
    )


admin.site.register(models.User, UserAdmin)
```

### ✅ Verification

```bash
docker compose run --rm app sh -c "python manage.py test"
docker compose run --rm app sh -c "flake8"
```

---

# Part 4: Wait for DB Command

## Step 4.1: Create Management Command Directory

```bash
mkdir -p app/core/management/commands
echo "" > app/core/management/__init__.py
echo "" > app/core/management/commands/__init__.py
```

## Step 4.2: Create Test for wait_for_db Command

Create `app/core/tests/test_commands.py`:

```python
"""
Test custom django management commands.
"""
from unittest.mock import patch
from psycopg2 import OperationalError as Psycopg2OpError
from django.core.management import call_command
from django.db.utils import OperationalError
from django.test import SimpleTestCase


@patch('core.management.commands.wait_for_db.Command.check')
class CommandTests(SimpleTestCase):
    """Test commands."""

    def test_wait_for_db_ready(self, patched_check):
        """Test waiting for database if database ready."""
        patched_check.return_value = True
        call_command('wait_for_db')
        patched_check.assert_called_once_with(databases=['default'])

    @patch('time.sleep')
    def test_wait_for_db_delay(self, patched_sleep, patched_check):
        """Test waiting for database when getting OperationalError."""
        patched_check.side_effect = [Psycopg2OpError] * 2 + \
            [OperationalError] * 3 + [True]
        call_command('wait_for_db')
        self.assertEqual(patched_check.call_count, 6)
        patched_check.assert_called_with(databases=['default'])
```

## Step 4.3: Create wait_for_db Command

Create `app/core/management/commands/wait_for_db.py`:

```python
"""
Django command to wait for the database to be available.
"""
import time
from psycopg2 import OperationalError as Psycopg2OpError
from django.db.utils import OperationalError
from django.core.management.base import BaseCommand


class Command(BaseCommand):
    """Django command to wait for database."""

    def handle(self, *args, **options):
        """Entrypoint for command."""
        self.stdout.write('Waiting for database...')
        db_up = False
        while db_up is False:
            try:
                self.check(databases=['default'])
                db_up = True
            except (Psycopg2OpError, OperationalError):
                self.stdout.write('Database unavailable, waiting 1 second...')
                time.sleep(1)

        self.stdout.write(self.style.SUCCESS('Database available!'))
```

### ✅ Verification

```bash
docker compose run --rm app sh -c "python manage.py test"
docker compose run --rm app sh -c "flake8"
```

---

# Part 5: User API

## Step 5.1: Create User App

```bash
docker compose run --rm app sh -c "python manage.py startapp user"

# Create tests directory
mkdir app/user/tests
echo "" > app/user/tests/__init__.py
rm app/user/tests.py
```

## Step 5.2: Update settings.py

Add to `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # ... existing apps
    'rest_framework',
    'rest_framework.authtoken',
    'user',
]
```

## Step 5.3: Create User Serializers

Create `app/user/serializers.py`:

```python
from django.contrib.auth import (get_user_model, authenticate,)
from django.utils.translation import gettext as _
from rest_framework import serializers


class UserSerializer(serializers.ModelSerializer):
    """Serializer for the user object."""

    class Meta:
        model = get_user_model()
        fields = ['email', 'password', 'name']
        extra_kwargs = {'password': {'write_only': True, 'min_length': 5}}

    def create(self, validated_data):
        """Create and return a user with encrypted password."""
        return get_user_model().objects.create_user(**validated_data)

    def update(self, instance, validated_data):
        """Update and return user."""
        password = validated_data.pop('password', None)
        user = super().update(instance, validated_data)
        if password:
            user.set_password(password)
            user.save()
        return user


class AuthTokenSerializer(serializers.Serializer):
    """Serializer for the user auth token."""
    email = serializers.EmailField()
    password = serializers.CharField(
        style={'input_type': 'password'},
        trim_whitespace=False,
    )

    def validate(self, attrs):
        """Validate and authenticate the user."""
        email = attrs.get('email')
        password = attrs.get('password')
        user = authenticate(
            request=self.context.get('request'),
            username=email,
            password=password,
        )
        if not user:
            msg = _('Unable to authenticate with provided credentials.')
            raise serializers.ValidationError(msg, code='authorization')
        attrs['user'] = user
        return attrs
```

## Step 5.4: Create User Views

Create `app/user/views.py`:

```python
"""
Views for the user API.
"""
from rest_framework import generics, authentication, permissions
from rest_framework.authtoken.views import ObtainAuthToken
from rest_framework.settings import api_settings
from user.serializers import UserSerializer, AuthTokenSerializer


class CreateUserView(generics.CreateAPIView):
    """Create a new user in the system."""
    serializer_class = UserSerializer


class CreateTokenView(ObtainAuthToken):
    """Create a new auth token for user."""
    serializer_class = AuthTokenSerializer
    renderer_classes = api_settings.DEFAULT_RENDERER_CLASSES


class ManageUserView(generics.RetrieveUpdateAPIView):
    """Manage the authenticated user."""
    serializer_class = UserSerializer
    authentication_classes = [authentication.TokenAuthentication]
    permission_classes = [permissions.IsAuthenticated]

    def get_object(self):
        """Retrieve and return the authenticated user."""
        return self.request.user
```

## Step 5.5: Create User URLs

Create `app/user/urls.py`:

```python
"""
URL mappings for the user API.
"""
from django.urls import path
from user import views

app_name = 'user'

urlpatterns = [
    path('create/', views.CreateUserView.as_view(), name='create'),
    path('token/', views.CreateTokenView.as_view(), name='token'),
    path('me/', views.ManageUserView.as_view(), name='me'),
]
```

## Step 5.6: Update Main URLs

Update `app/app/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/user/', include('user.urls')),
]
```

### ✅ Verification

```bash
docker compose run --rm app sh -c "python manage.py test"
docker compose run --rm app sh -c "flake8"
```

---

# Part 6: Recipe API

## Step 6.1: Create Recipe App

```bash
docker compose run --rm app sh -c "python manage.py startapp recipe"

mkdir app/recipe/tests
echo "" > app/recipe/tests/__init__.py
rm app/recipe/tests.py
```

## Step 6.2: Update settings.py

Add `'recipe',` to `INSTALLED_APPS`.

## Step 6.3: Add Recipe Model to core/models.py

Add to `app/core/models.py`:

```python
from django.conf import settings

class Recipe(models.Model):
    """Recipe object."""
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
    title = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    time_minutes = models.IntegerField()
    price = models.DecimalField(max_digits=5, decimal_places=2)
    link = models.CharField(max_length=255, blank=True)

    def __str__(self):
        return self.title
```

## Step 6.4: Create Migration

```bash
docker compose run --rm app sh -c "python manage.py makemigrations"
```

## Step 6.5: Register Recipe in Admin

Update `app/core/admin.py`, add:

```python
admin.site.register(models.Recipe)
```

### ✅ Verification

```bash
docker compose run --rm app sh -c "python manage.py test"
docker compose run --rm app sh -c "flake8"
```

---

# Part 7: Tags & Ingredients API

## Step 7.1: Add Tag and Ingredient Models

Add to `app/core/models.py`:

```python
class Tag(models.Model):
    """Tag for filtering recipes."""
    name = models.CharField(max_length=255)
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )

    def __str__(self):
        return self.name


class Ingredient(models.Model):
    """Ingredient for recipes."""
    name = models.CharField(max_length=255)
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )

    def __str__(self):
        return self.name
```

Update Recipe model to include:

```python
tags = models.ManyToManyField('Tag')
ingredients = models.ManyToManyField('Ingredient')
```

## Step 7.2: Create Migration

```bash
docker compose run --rm app sh -c "python manage.py makemigrations"
```

### ✅ Verification

```bash
docker compose run --rm app sh -c "python manage.py test"
docker compose run --rm app sh -c "flake8"
```

---

# Part 8: Image Upload

## Step 8.1: Update Recipe Model for Images

Add to `app/core/models.py` at the top:

```python
import uuid
import os

def recipe_image_file_path(instance, filename):
    """Generate file path for new recipe image."""
    ext = os.path.splitext(filename)[1]
    filename = f'{uuid.uuid4()}{ext}'
    return os.path.join('uploads', 'recipe', filename)
```

Add to Recipe model:

```python
image = models.ImageField(null=True, upload_to=recipe_image_file_path)
```

## Step 8.2: Create Migration

```bash
docker compose run --rm app sh -c "python manage.py makemigrations"
```

### ✅ Verification

```bash
docker compose run --rm app sh -c "python manage.py test"
docker compose run --rm app sh -c "flake8"
```

---

# Part 9: API Documentation

## Step 9.1: Update settings.py

Add to `INSTALLED_APPS`:

```python
'drf_spectacular',
```

Add REST Framework settings:

```python
REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

SPECTACULAR_SETTINGS = {
    'COMPONENT_SPLIT_REQUEST': True,
}
```

## Step 9.2: Update Main URLs

Update `app/app/urls.py`:

```python
from drf_spectacular.views import (
    SpectacularAPIView,
    SpectacularSwaggerView,
)
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/schema/', SpectacularAPIView.as_view(), name='api-schema'),
    path(
        'api/docs/',
        SpectacularSwaggerView.as_view(url_name='api-schema'),
        name='api-docs',
    ),
    path('api/user/', include('user.urls')),
    path('api/recipe/', include('recipe.urls')),
]

if settings.DEBUG:
    urlpatterns += static(
        settings.MEDIA_URL,
        document_root=settings.MEDIA_ROOT,
    )
```

### ✅ Final Verification

```bash
# Run all tests
docker compose run --rm app sh -c "python manage.py test"

# Run flake8 linting
docker compose run --rm app sh -c "flake8"

# Start the application
docker compose up
```

---

## 🚀 Running the Application

```bash
# Build and start
docker compose up --build

# Run in background
docker compose up -d

# Stop
docker compose down

# View logs
docker compose logs -f app
```

## 📚 API Endpoints

| Endpoint                    | Method               | Description           |
| --------------------------- | -------------------- | --------------------- |
| `/api/docs/`                | GET                  | Swagger Documentation |
| `/api/user/create/`         | POST                 | Create new user       |
| `/api/user/token/`          | POST                 | Get auth token        |
| `/api/user/me/`             | GET/PATCH            | Manage profile        |
| `/api/recipe/recipes/`      | GET/POST             | List/Create recipes   |
| `/api/recipe/recipes/{id}/` | GET/PUT/PATCH/DELETE | Recipe detail         |
| `/api/recipe/tags/`         | GET                  | List tags             |
| `/api/recipe/ingredients/`  | GET                  | List ingredients      |

## 🧪 Testing Commands

```bash
# Run all tests
docker compose run --rm app sh -c "python manage.py test"

# Run specific app tests
docker compose run --rm app sh -c "python manage.py test core"
docker compose run --rm app sh -c "python manage.py test user"
docker compose run --rm app sh -c "python manage.py test recipe"

# Run with verbosity
docker compose run --rm app sh -c "python manage.py test --verbosity 2"

# Run flake8
docker compose run --rm app sh -c "flake8"
```

## 🔐 Create Superuser

```bash
docker compose run --rm app sh -c "python manage.py createsuperuser"
```

---

## 📝 License

This project is open source and available under the MIT License.
