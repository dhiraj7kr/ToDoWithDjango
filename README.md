# ToDoWithDjango
Here’s a tree structure showing how the files should be organized for your Django To-Do application:

```
todo_project/
├── manage.py
├── db.sqlite3
├── todo_project/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   ├── wsgi.py
│   └── __pycache__/
├── tasks/
│   ├── migrations/
│   │   ├── __init__.py
│   │   ├── 0001_initial.py  # Auto-generated after migrations
│   │   └── __pycache__/
│   ├── templates/
│   │   ├── base.html
│   │   └── tasks/
│   │       ├── task_list.html
│   │       ├── add_task.html
│   │       ├── edit_task.html
│   │       └── trash.html
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── forms.py
│   ├── models.py
│   ├── tests.py
│   ├── views.py
│   └── urls.py
└── static/  # Optional folder for static assets like CSS, JS, images
    ├── css/
    │   └── styles.css  # Custom styles if needed
    ├── js/
    └── images/
```

---

### Explanation of Each File:

1. **`manage.py`**: Entry point for Django commands like running the server and migrations.
2. **`todo_project/`**: The main project directory containing:
   - `settings.py`: Configuration for the project (e.g., database, installed apps).
   - `urls.py`: Root URL configuration to include app-specific URLs.
3. **`tasks/`**: The app directory containing:
   - `models.py`: Defines the `Task` model.
   - `views.py`: Contains logic for task management (add, edit, delete, etc.).
   - `urls.py`: App-specific URLs.
   - `forms.py`: Contains the `TaskForm` class for handling form rendering and validation.
   - `templates/tasks/`: Contains HTML templates specific to the app.
   - `migrations/`: Auto-generated files for database schema changes.
4. **`templates/`**:
   - `base.html`: Shared layout for all pages.
   - Other task-related templates under `templates/tasks/`.
5. **`static/`**: (Optional) Contains static assets like CSS or JS files for custom styling and interactivity.
6. **`db.sqlite3`**: The SQLite database file generated after running migrations.

---

Here is a step-by-step guide and example code for creating a To-Do web application with Django, incorporating the requested features. This guide will cover the following:

1. **Setup and Configuration**
2. **Models**
3. **Views and URLs**
4. **Templates**
5. **User Authentication**
6. **Task Management**
7. **Trash Feature**
8. **Notifications**
9. **Styling and Enhancements**

---

## 1. **Setup and Configuration**

### Install Django:
```bash
pip install django
```

### Create a Django project:
```bash
django-admin startproject todo_project
cd todo_project
```

### Create an app for tasks:
```bash
python manage.py startapp tasks
```

### Add the app to `INSTALLED_APPS` in `todo_project/settings.py`:
```python
INSTALLED_APPS = [
    # Default apps
    'tasks',  # Our custom app
    'django.contrib.sites',  # Required for notifications (optional)
]
```

### Setup Database (SQLite by default):
No changes are needed for SQLite, which is already configured in `settings.py`.

---

## 2. **Models**

Create models for `Task` in `tasks/models.py`:

```python
from django.db import models
from django.contrib.auth.models import User

class Task(models.Model):
    PRIORITY_CHOICES = [
        ('Very Important', 'Very Important'),
        ('Important', 'Important'),
    ]
    REPEAT_CHOICES = [
        ('None', 'None'),
        ('Daily', 'Daily'),
        ('Weekly', 'Weekly'),
        ('Monthly', 'Monthly'),
        ('Yearly', 'Yearly'),
    ]

    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name="tasks")
    title = models.CharField(max_length=255)
    description = models.TextField(blank=True, null=True)
    due_date = models.DateTimeField()
    repeat = models.CharField(max_length=10, choices=REPEAT_CHOICES, default='None')
    priority = models.CharField(max_length=15, choices=PRIORITY_CHOICES)
    is_deleted = models.BooleanField(default=False)

    def __str__(self):
        return self.title
```

Run migrations:
```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 3. **Views and URLs**

Create views for CRUD operations in `tasks/views.py`:

### Add Task View:
```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from .models import Task
from .forms import TaskForm

@login_required
def add_task(request):
    if request.method == "POST":
        form = TaskForm(request.POST)
        if form.is_valid():
            task = form.save(commit=False)
            task.user = request.user
            task.save()
            return redirect('task_list')
    else:
        form = TaskForm()
    return render(request, 'tasks/add_task.html', {'form': form})
```

### Edit Task View:
```python
@login_required
def edit_task(request, task_id):
    task = get_object_or_404(Task, id=task_id, user=request.user)
    if request.method == "POST":
        form = TaskForm(request.POST, instance=task)
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task)
    return render(request, 'tasks/edit_task.html', {'form': form})
```

### Delete Task (Move to Trash):
```python
@login_required
def delete_task(request, task_id):
    task = get_object_or_404(Task, id=task_id, user=request.user)
    task.is_deleted = True
    task.save()
    # Add logic for notification here
    return redirect('task_list')
```

### Task List:
```python
@login_required
def task_list(request):
    tasks = Task.objects.filter(user=request.user, is_deleted=False)
    return render(request, 'tasks/task_list.html', {'tasks': tasks})
```

### Trash View:
```python
@login_required
def trash(request):
    tasks = Task.objects.filter(user=request.user, is_deleted=True)
    return render(request, 'tasks/trash.html', {'tasks': tasks})
```

---

## 4. **URLs**

Add URLs in `tasks/urls.py`:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.task_list, name='task_list'),
    path('add/', views.add_task, name='add_task'),
    path('edit/<int:task_id>/', views.edit_task, name='edit_task'),
    path('delete/<int:task_id>/', views.delete_task, name='delete_task'),
    path('trash/', views.trash, name='trash'),
]
```

Include these in the project’s `urls.py`:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('tasks.urls')),
]
```

---

## 5. **Templates**

### Base Template (`base.html`):
```html
<!DOCTYPE html>
<html>
<head>
    <title>To-Do App</title>
</head>
<body>
    <header>
        <nav>
            <a href="{% url 'task_list' %}">Home</a>
            <a href="{% url 'add_task' %}">Add Task</a>
            <a href="{% url 'trash' %}">Trash</a>
        </nav>
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```

### Task List (`task_list.html`):
```html
{% extends 'base.html' %}
{% block content %}
<h1>Your Tasks</h1>
<ul>
    {% for task in tasks %}
    <li>
        {{ task.title }} - {{ task.due_date }}
        <a href="{% url 'edit_task' task.id %}">Edit</a>
        <a href="{% url 'delete_task' task.id %}">Delete</a>
    </li>
    {% endfor %}
</ul>
{% endblock %}
```

---

## 6. **Forms**

Create a form for Task in `tasks/forms.py`:
```python
from django import forms
from .models import Task

class TaskForm(forms.ModelForm):
    class Meta:
        model = Task
        fields = ['title', 'description', 'due_date', 'repeat', 'priority']
```

---

## 7. **Notifications**

For notifications, you can use Django's message framework. Configure it in `settings.py`:
```python
from django.contrib.messages import constants as messages

MESSAGE_TAGS = {
    messages.ERROR: 'danger',
}

INSTALLED_APPS += [
    'django.contrib.messages',
]

MIDDLEWARE += [
    'django.contrib.messages.middleware.MessageMiddleware',
]
```

In `delete_task` view, add:
```python
from django.contrib import messages

@login_required
def delete_task(request, task_id):
    task = get_object_or_404(Task, id=task_id, user=request.user)
    task.is_deleted = True
    task.save()
    messages.success(request, f'Task "{task.title}" has been moved to trash.')
    return redirect('task_list')
```

In templates, display messages:
```html
{% if messages %}
<ul>
    {% for message in messages %}
    <li>{{ message }}</li>
    {% endfor %}
</ul>
{% endif %}
```

---

## 8. **Styling and Enhancements**

For a responsive UI, integrate **Bootstrap** or **TailwindCSS**.

---

Let me know if you'd like me to expand on any part of the setup or implementation!
