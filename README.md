# Advanced Django Cheat Sheet

Be aware it's not an exhaustive list.
If you have ideas, correction or recommendation do not hesitate.

## Sections
- [Preparing environnement](#preparing-environnement)
- [Creating a Django project](#creating-a-django-project)
- [Creating a Django app](#creating-a-django-app)
- [Custom User](#custom-user)
	- [Custom user model](#custom-user-model)
	- [Custom user forms](#custom-user-forms)
	- [Custom user admin](#custom-user-admin)
	- [Superuser](#superuser)
- [Migration](#migration)
	- [makemigration and migrate](#makemigration-and-migrate)
	- [Fake initial migration](#fake-initial-migration)
- [Models](#models)
	- [Model style ordering](#model-style-ordering)
	- [Model and field names](#model-and-field-names)
	- [Choices](#choices)
	- [Blank and Null fields](#blank-and-null-fields)
	- [Meta class](#meta-class)
	- [The str method](#the-str-method)
	- [The get_absolute_url method](#the-get_absolute_url-method)
	- [UniqueConstraint](#uniqueconstraint)
	- [Models: Further Reading](#models-further-reading)
- [Model Managers](#model-managers)
	- [Giving a custom name to the default manager](#giving-a-custom-name-to-the-default-manager)
	- [Creating custom managers](#creating-custom-managers)
	- [Modifying a manager's initial QuerySet](#modifying-a-managers-initial-queryset)
- [Model registration in admin](#model-registration-in-admin)
- [Django Signals](#django-signals)
- [Queries and QuerySet](#queries-and-queryset)
	- [Using Q objects for complex queries](#using-q-objects-for-complex-queries)
	- [Aggregation](#aggregation)
	- [Latest element in QuerySet](#latest-element-in-queryset)
	- [Union of QuerySets](#union-of-querysets)
	- [Fixing the N+1 queries problem](#fixing-the-n1-queries-problem)
	- [Performing Raw SQL Queries](#performing-raw-sql-queries)
- [View](#view)
	- [Function-based views (FBVs)](#function-based-views-fbvs)
	- [Class-based views (CBVs)](#class-based-views-cbvs)
	- [Redirect from view](#redirect-from-view)
	- [View: Further reading](#view-further-reading)
- [Routing](#routing)
- [Authentication](#authentication)
	- [Authentication views and URLs](#authentication-views-and-urls)
	- [Signup](#signup)
	- [Password reset](#password-reset)
	- [OAuth](#oauth)
	- [Authentication : Further reading](#authentication-further-reading)
- [Custom Permissions](#custom-permissions)
- [Middleware](#middleware)
	- [Custom middleware](#custom-middleware)
- [Form and Form Validation](#form-and-form-validation)
	- [Form](#form)
	- [Selecting the fields to use](#selecting-the-fields-to-use)
	- [Form template](#form-template)
	- [Custom form field validators](#custom-form-field-validators)
	- [clean()](#clean)
	- [clean_field_name()](#clean_field_name)
- [Template](#template)
	- [Template inheritance and inclusion](#template-inheritance-and-inclusion)
	- [Common template tags](#common-template-tags)
- [Performance](#performance)
	- [django-debug-toolbar](#django-debug-toolbar)
	- [select_related and prefetch_related](#select_related-and-prefetch_related)
	- [Indexes](#indexes)
	- [Caching](#caching)
- [Security](#security)
	- [Admin hardening](#admin-hardening)
	- [Cross site request forgery (CSRF) protection](#cross-site-request-forgery-csrf-protection)
	- [Enforcing SSL HTTPS](#enforcing-ssl-https)
	- [ALLOWED_HOSTS](#allowed_hosts)
- [Further Reading](#further-eading)

## Preparing enviro­nnement
- Create project folder and navigate to it.
```
mkdir projec­t_name && cd $_
```

- Create Python virtual env.
```
python -m venv env_name
```
- Activate virtual env.
(Replace "­bin­" by "­Scr­ipt­s" in Windows).
```
source env_na­me­\bin­\ac­tivate
```
- Deactivate virtual env.
```
deactivate
```
- Install Django.
```
pip install django~=4.2.2
```
- Create requir­ements file.
```
pip freeze > requir­eme­nts.txt
```
- Install all required depend­encies based on your pip freeze command.
```
pip install -r requir­eme­nts.txt
```

## Creating a Django project
- Starting a new Django project.
A config directory wil be created in your current directory.
```
django-admin startproject config .
```
- Running the server
```
python manage.py runserver
```

## Creating a Django app
- Creating an `my_app` directory and all default files/f­olders inside.
```
python manage.py startapp my_app
```
- Adding the app to settings.py.
```
INSTAL­LED­_APPS = [
 'my_app',
 ...
```
- Adding app urls into the urls.py from project folder.
```
urlpat­terns = [
 path('admin/', admin.s­it­e.u­rls),
 path('my_app/', include('my_app.urls')),
]
```
## Custom User
### Custom User Model
[Django documentation: Using a custom user model when starting a project](https://docs.djangoproject.com/en/stable/topics/auth/customizing/#using-a-custom-user-model-when-starting-a-project)  
1. Create a `CustomUser` model  
	The `CustomUser` model will live within its own app (for example, 'accounts').
	```python
	# accounts/models.py
	from django.contrib.auth.models import AbstractUser
	from django.db import models
	
	class CustomUser(AbstractUser):
		pass
	```
2. Update `settings.py`
	- Add the 'accounts' app to the `INSTALLED_APPS` section
	- Add a `AUTH_USER_MODEL` config:
	```
	AUTH_USER_MODEL = "accounts.CustomUser"
	```
	- Create migrations file
	```
	python manage.py makemigrations accounts
	```
	- Migrate
### Custom User Forms
Updating the built-in forms to point to the custom user model instead of `User`.    

```python
# accounts/forms.py
from django.contrib.auth import get_user_model
from django.contrib.auth.forms import UserCreationForm, UserChangeForm

class CustomUserCreationForm(UserCreationForm):
	class Meta:
		model = get_user_model()
		fields = (
			"email",
			"username",
		)
	

class CustomUserChangeForm(UserChangeForm):
	class Meta:
		model = get_user_model()
		fields = (
			"email",
			"username",
		)
```

### Custom User Admin
Extending the existing `UserAdmin` into `CustomUserAdmin`.    

```python
# accounts/admin.py
from django.contrib import admin
from django.contrib.auth import get_user_model
from django.contrib.auth.admin import UserAdmin

from .forms import CustomUserCreationForm, CustomUserChangeForm

CustomUser = get_user_model()

class CustomUserAdmin(UserAdmin):
	add_form = CustomUserCreationForm
	form = CustomUserChangeForm
	model = CustomUser
	list_display = [
		"email",
		"username",
		"is_superuser",
	]
```

### Superuser
```
python manage.py createsuperuser
```

## Migration
### makemigration and migrate
[Migrations in the Django Doc](https://docs.djangoproject.com/en/stable/topics/migrations/)

**makemigrations**: This command generates migration files based on the changes detected in your models.
It compares the current state of your models with the migration files already created and determines the SQL commands required to propagate the changes to your database schema.
```
python manage.py makemigrations
```

**migrate**: This command applies the migration files to your database, executing the SQL commands generated by `makemigrations`.
```
python manage.py migrate
```
This will update your database schema with the changes made to your models.

### Fake initial migration
 In Django, a "fake initial migration" refers to a concept where you mark a migration as applied without actually executing the database schema changes associated with that migration.
It allows you to synchronize the state of the migrations with the database without performing any database modifications.
 ```
python manage.py migrate --fake-initial
```
It's important to note that **faking the initial migration assumes that the existing database schema matches what the initial migration would have created**.

## Models
[Django Documentation: Models](https://docs.djangoproject.com/en/stable/topics/db/models/#module-django.db.models)  

### Model Style Ordering
- Choices
- Database fields
- Custom manager attributes
- Meta class
- def `__str__()`
- def `save()`
- def `get_absolute_url()`
- Custom methods

### Model and Field Names
```python
# my_book_app/models.py
from django.db import models

class Book(models.Model):
	title = models.CharField(max_length=100)
```
Models represents a single object and should always be Capitalized and singular (Book, not Books).

Fields should all be snake_case, not camelCase.

### Choices
If choices are defined for a given model field, define each choice as a tuple of tuples, with an all-uppercase name as a class attribute on the model ([source](https://learndjango.com/tutorials/django-best-practices-models)).
```python
# my_book_app/models.py
from django.db import models

class Book(models.Model):
	BOOK_CATEGORY = (
		("FICTION", "A fiction book"),
		("NON_FICTION", "A non-fiction book")
	)
	title = models.CharField(max_length=100)
	book_type = models.CharField(
		choices=BOOK_CATEGORY,
		max_lenght=100,
		verbose_name="type of book",
	)
```

### Blank and Null Fields
 - **Null**: Database-related. Defines if a given database column will accept null values or not.
 - **Blank**: Validation-related. It will be used during forms validation, when calling `form.is_valid()`.
 
Do not use null with string-based fields like `CharField` or `TextField` as this leads to two possible values for "no data".   The Django convention is instead to use the empty string "", not `NULL` ([source](https://simpleisbetterthancomplex.com/tips/2018/02/10/django-tip-22-designing-better-models.html). 

### Meta class
[Django Documentation: Meta class](https://docs.djangoproject.com/en/stable/ref/models/options/)

An example, using `[indexes](#indexes)`, `ordering`, `verbose_name` and `verbose_name_plural`.  
(Don't order results if you don't need to. There can be [performance hit to ordering results](https://docs.djangoproject.com/en/dev/topics/db/optimization/#don-t-order-results-if-you-don-t-care).)  
```python
# my_book_app/models.py
from django.db import models

class Book(models.Model):
	BOOK_CATEGORY = (
		("FICTION", "A fiction book"),
		("NON_FICTION", "A non-fiction book")
	)
	title = models.CharField(max_length=100)
	book_type = models.CharField(
		choices=BOOK_CATEGORY,
		max_lenght=100,
		verbose_name="type of book",
	)
	
	class Meta:
		indexes = [models.Index(fields=["title"])]
		ordering = ["-title"]
		verbose_name = "book"
		verbose_name_plural = "books"
```


### The str Method
[Django Documentation: __str__()](https://docs.djangoproject.com/en/stable/ref/models/instances/#str)  
The str method defines a string representation, a more descriptive name/title, for any of our objects that is displayed in the Django admin site and in the Django shell.
```python
# my_book_app/models.py
from django.db import models

class Book(models.Model):
	BOOK_CATEGORY = (
		("FICTION", "A fiction book"),
		("NON_FICTION", "A non-fiction book")
	)
	title = models.CharField(max_length=100)
	book_type = models.CharField(
		choices=BOOK_CATEGORY,
		max_lenght=100,
		verbose_name="type of book",
	)
	
	class Meta:
		indexes = [models.Index(fields=["title"])]
		ordering = ["-title"]
		verbose_name = "book"
		verbose_name_plural = "books"
	
	def __str__(self):
		return self.title
```

### The get_absolute_url Method
[Django Documentation: get_absolute_url()](https://docs.djangoproject.com/en/stable/ref/models/instances/#get-absolute-url)  
The `get_absolute_url` method sets a canonical URL for the model.
```python
# my_book_app/models.py
from django.db import models

class Book(models.Model):
	BOOK_CATEGORY = (
		("FICTION", "A fiction book"),
		("NON_FICTION", "A non-fiction book")
	)
	title = models.CharField(max_length=100)
	book_type = models.CharField(
		choices=BOOK_CATEGORY,
		max_lenght=100,
		verbose_name="type of book",
	)
	
	class Meta:
		indexes = [models.Index(fields=["title"])]
		ordering = ["-title"]
		verbose_name = "book"
		verbose_name_plural = "books"
	
	def __str__(self):
		return self.title
		
	def get_absolute_url(self):
		return reverse("book_detail", args=[str(self.id)])
```
Using the `get_absolute_url` in our templates:
```
<a href="{{ object.get_absolute_url }}/">{{ object.title }}</a>
```

### UniqueConstraint
[Django Documentation: uniqueConstraint](https://docs.djangoproject.com/en/stable/ref/models/constraints/#uniqueconstraint)

Use `UniqueConstraint` when you want to enforce uniqueness on a combination of fields or need additional functionality like custom constraint names or conditional constraints

```python
class Booking(models.Model):
    room = models.ForeignKey(Room, on_delete=models.CASCADE)
    date = models.DateField()

    class Meta:
        constraints = [
            models.UniqueConstraint(fields=['room', 'date'], name='unique_booking')
        ]
```

### Models: Further reading
- [LearnDjango: Django Best Practices: Models (2022)](https://learndjango.com/tutorials/django-best-practices-models)  
- [Simple is better than complex: Designing Better Models (2018)](https://simpleisbetterthancomplex.com/tips/2018/02/10/django-tip-22-designing-better-models.html)

## Model Managers
[Django Documentation: Managers](https://docs.djangoproject.com/en/stable/topics/db/managers/)  
### Giving a custom name to the default manager

```python
class Author(models.Model):
    ...
    authors = models.Manager() //now the default manager is named as authors
```
All the operation on the student database table have to be done using the “authors” manager
```
Author.authors.filter(...)
```
### Creating custom managers
[Django Documentation: Custom managers](https://docs.djangoproject.com/en/stable/topics/db/managers/#custom-managers)
```python
from django.db import models
from django.db.models.functions import Coalesce


class PollManager(models.Manager):
    def with_counts(self):
        return self.annotate(num_responses=Coalesce(models.Count("response"), 0))


class OpinionPoll(models.Model):
    question = models.CharField(max_length=200)
    objects = PollManager()


class Response(models.Model):
    poll = models.ForeignKey(OpinionPoll, on_delete=models.CASCADE)
    # ...
```

> If you use custom Manager objects, take note that the first Manager Django encounters (in the order in which they’re defined in the model) has a special status. Django interprets the first Manager defined in a class as the “default” Manager, and several parts of Django (including dumpdata) will use that Manager exclusively for that model. As a result, it’s a good idea to be careful in your choice of default manager in order to avoid a situation where overriding get_queryset() results in an inability to retrieve objects you’d like to work with.

### Modifying a manager’s initial QuerySet
[Django Documenation: Modifying a managers's initial QuerySet](https://docs.djangoproject.com/en/stable/topics/db/managers/#modifying-a-manager-s-initial-queryset)
```
# First, define the Manager subclass.
class DahlBookManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(author="Roald Dahl")


# Then hook it into the Book model explicitly.
class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=50)

    objects = models.Manager()  # The default manager.
    dahl_objects = DahlBookManager()  # The Dahl-specific manager.
```
With this sample model, `Book.objects.all()` will return all books in the database, but `Book.dahl_objects.all()` will only return the ones written by Roald Dahl.
## Model registration in admin
[Django doc: ModelAdmin objects](https://docs.djangoproject.com/en/stable/ref/contrib/admin/#modeladmin-objects)

Model registration in Django's admin interface is the process of making your models accessible through the admin site.

```python
from django.contrib import admin
from .models import Author

admin.site.register(Author)
```

- Customizing the display of a model

```python
from django.contrib import admin
from .models import Book

class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'publication_date')

admin.site.register(Book, BookAdmin)
```

- Adding a search field

```python
from django.contrib import admin
from .models import Publisher

class PublisherAdmin(admin.ModelAdmin):
    search_fields = ['name']

admin.site.register(Publisher, PublisherAdmin)
```

- Adding filters

```python
from django.contrib import admin
from .models import Category

class CategoryAdmin(admin.ModelAdmin):
    list_filter = ('is_active',)

admin.site.register(Category, CategoryAdmin)
```

- Inline formsets

```python
from django.contrib import admin
from .models import Order, OrderItem

class OrderItemInline(admin.TabularInline):
    model = OrderItem
    extra = 1

class OrderAdmin(admin.ModelAdmin):
    inlines = [OrderItemInline]

admin.site.register(Order, OrderAdmin)
```

## Django Signals
- **pre_save**:  
[Django Doc: pre_save](https://docs.djangoproject.com/en/stable/ref/signals/#pre-save)  
Using a `pre_save` signal is required to execute code related to another part of your application *prior* to saving the object in the database.

```python
from django.db.models.signals import pre_save
from django.dispatch import receiver

@receiver(pre_save, sender=NewOrder)
def validate_order(sender, instance, **kwargs):
    stock_item = Stock.objects.get(id=instance.stock_item.id)

    if instance.quantity > stock_item.quantity:
        raise Exception("Insufficient stock quantity.")
```

- **post_save**:  
[Django Doc: post_save](https://docs.djangoproject.com/en/stable/ref/signals/#post-save)  
Using a `post_save` signal is required to execute code related to another part of your application *after* the object is saved to the database.

```python
from django.db.models.signals import post_save
from django.dispatch import receiver


@receiver(post_save, sender=NewOrder)
def remove_from_inventory(sender, instance, **kwargs):
    stock_item = Inventory.objects.get(id=instance.stock_item.id)
    stock_item.quantity = stock_item.quantity - instance.quantity

    stock_item.save()
```

- **pre_delete**:  
[Django Doc: pre_delete](https://docs.djangoproject.com/en/stable/ref/signals/#pre-delete)  
Using a `pre_delete` signal is necessary to execute code related to another part of your application *before* the deletion event of an object occurs.

```python
from django.db.models.signals import pre_delete
from django.dispatch import receiver


@receiver(pre_delete, sender=Book)
def pre_delete_book(sender, **kwargs):
    print("You are about to delete a book")
```

- **post_delete**
[Django Doc: post_delete](https://docs.djangoproject.com/en/stable/ref/signals/#post-delete)
Using a `post_delete` signal is necessary to execute code related to another part of your application *after* the deletion event of an object occurs.

```python
@receiver(post_delete, sender=Book)
def delete_book(sender, **kwargs):
    print("You have just deleted a book")
```

- **m2m_changed**
[Django Doc: m2m_changed](https://docs.djangoproject.com/en/stable/ref/signals/#m2m-changed)
To send a Django signal when a `ManyToManyField` is changed on a model instance.

Consider this model:
```python
class Student(models.Model):
    # ...


class Course(models.Model):
    students = models.ManyToManyField(Student)
```
m2m_changed signal:
```python
from django.db.models.signals import m2m_changed

def my_signal_name(sender, instance, **kwargs):
    students = instance.students.all()
    # ...

m2m_changed.connect(my_signal_name, sender=Course.students.through)
```
## Queries and QuerySet
[Django Documentation: Making queries](https://docs.djangoproject.com/en/stable/topics/db/queries/#making-queries)
### Using Q objects for complex queries
[Django Documentation: Q objects](https://docs.djangoproject.com/en/stable/topics/db/queries/#complex-lookups-with-q-objects)

Q objects can be combined using the `&` (AND) and `|` (OR) operators
```
Inventory.objects.filter(
    Q(quantity__lt=10) &
    Q(next_shipping__gt=datetime.datetime.today()+datetime.timedelta(days=10))
)
```

```
Inventory.objects.filter(
    Q(name__icontains="robot") |
    Q(title__icontains="vacuum")
```
### Aggregation
[Django documenation: Aggregation](https://docs.djangoproject.com/en/stable/topics/db/aggregation/#aggregation)

In Django, aggregation allows you to perform calculations such as counting, summing, averaging, finding the maximum or minimum value, and more, on a specific field or set of fields in a queryset.

```python
from django.db.models import Sum

total_ratings = Movies.objects.aggregate(ratings_sum=Sum('ratings_count'))
```

**Utilizing Aggregation in Views and Templates**

In the view:
```python
from django.shortcuts import render

def example(request):
    data = Movies.objects.aggregate(ratings_sum=Sum('ratings_count'))
    return render(request, 'index.html', {'data': data})
```

In the template:
```
<p>Total Ratings: {{ data.ratings_sum }}</p>
```
### Latest element in QuerySet
[Django Documentation: latest()](https://docs.djangoproject.com/en/stable/ref/models/querysets/#latest)

This example returns the latest Entry in the table, according to the pub_date field:
```Python
Entry.objects.latest("pub_date")
```
You can also choose the latest based on several fields.
For example, to select the Entry with the earliest `expire_date` when two entries have the same pub_date:
```python
Entry.objects.latest("pub_date", "-expire_date")
```
The negative sign in `'-expire_date'` means to sort `expire_date` in descending order. Since `latest()` gets the last result, the `Entry` with the earliest `expire_date` is selected.
### Union of QuerySets
[union() in the Django Doc](https://docs.djangoproject.com/en/stable/ref/models/querysets/#union)

Uses SQL’s `UNION` operator to combine the results of two or more QuerySets.
For example:
```
>>> qs1.union(qs2, qs3)
```

`union()` return model instances of the type of the first QuerySet even if the arguments are QuerySets of other models.
Passing different models works **as long as the SELECT list is the same in all QuerySets** (at least the types, the names don’t matter as long as the types are in the same order).
In such cases, you must use the column names from the first QuerySet in QuerySet methods applied to the resulting QuerySet.
For example:
```
>>> qs1 = Author.objects.values_list("name")
>>> qs2 = Entry.objects.values_list("headline")
>>> qs1.union(qs2).order_by("name")
```
In addition, only `LIMIT`, `OFFSET`, `COUNT(*)`, `ORDER BY`, and specifying columns (i.e. slicing, count(), exists(), order_by(), and values()/values_list()) are allowed on the resulting QuerySet.
### Fixing the N+1 Queries Problem
See [select_related and prefetch_related](#select_related-and-prefetch_related)

### Performing raw SQL queries
[Django Documentation: Performing raw SQL queries](https://docs.djangoproject.com/en/stable/topics/db/sql/#performing-raw-sql-queries)

```python
from django.db import models


class Project(models.Model):
    title = models.CharField(max_length=70)
```

```
Project.objects.raw('SELECT id, title FROM myapp_project')
```

**Custom sql or raw queries sould be both used with extrement caution since they could open up a vulnerability to SQL injection**.

## View

### Function-based views (FBVs)  
[Django docucmentation: Writing views](https://docs.djangoproject.com/en/stable/topics/http/views/)

From [Django Views - The Right Way](https://spookylukey.github.io/django-views-the-right-way/the-pattern.html#the-explanation): Why `TemplateResponse` over `render` ?
>The issue with just using render is that you get a plain HttpResponse object back that has no memory that it ever came from a template. Sometimes, however, it is useful to have functions return a value that does remember what it’s “made of” — something that stores the template it is from, and the context. This can be really useful in testing, but also if we want to something outside of our view function (such as decorators or middleware) to check or even change what’s in the response before it finally gets ‘rendered’ and sent to the user.

```python
from django.template.response import TemplateResponse
from django.shortcuts import get_object_or_404, redirect

from .forms import TaskForm, ConfirmForm
from .models import Task


def task_list_view(request):
    return TemplateResponse(request, 'task_list.html', {
        'tasks': Task.objects.all(),
    })


def task_create_view(request):
    if request.method == 'POST':
        form = TaskForm(data=request.POST)
        if form.is_valid():
            task = form.save()
            return redirect('task_detail', pk=task.pk)

    return TemplateResponse(request, 'task_create.html', {
        'form': TaskForm(),
    })


def task_detail_view(request, pk):
    task = get_object_or_404(Task, pk=pk)

    return TemplateResponse(request, 'task_detail.html', {
        'task': task,
    })


def task_update_view(request, pk):
    task = get_object_or_404(Task, pk=pk)

    if request.method == 'POST':
        form = TaskForm(instance=task, data=request.POST)
        if form.is_valid():
            form.save()
            return redirect('task_detail', pk=task.pk)

    return TemplateResponse(request, 'task_edit.html', {
        'task': task,
        'form': TaskForm(instance=task),
    })


def task_delete_view(request, pk):
    task = get_object_or_404(Task, pk=pk)

    if request.method == 'POST':
        form = ConfirmForm(data=request.POST)
        if form.is_valid():
            task.delete()
            return redirect('task_list')

    return TemplateResponse(request, 'task_delete.html', {
        'task': task,
        'form': ConfirmForm(),
    })
```

### Class-based views (CBVs)
[Django Documentation : Class-based views](https://docs.djangoproject.com/en/stable/topics/class-based-views/)
```python
from django.views.generic import ListView, DetailView
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy

from .models import Task


class TaskListView(ListView):
    model = Task
    template_name = "task_list.html"


class BlogDetailView(DetailView):
    model = Task
    template_name = "task_detail.html"


class TaskCreateView(CreateView):
    model = Task
    template_name = "task_create.html"
    fields = ["name", "body", "author"]


class TaskUpdateView(UpdateView):
    model = Task
    template_name = "task_edit.html"
    fields = ["name", "body"]


class TaskDeleteView(DeleteView):
    model = Task
    template_name = "task_delete.html"
    success_url = reverse_lazy("task_list")
```

### Redirect from view:  
[Django Documentation: redirect()](https://docs.djangoproject.com/en/stable/topics/http/shortcuts/#redirect)
```python
from django.shortcuts import redirect

# Using the redirect() function by passing an object:
def my_view(request):
    ...
    obj = MyModel.objects.get(...)
    return redirect(obj)

# Using the redirect() function by passing the name of a view
# and optionally some positional or keyword arguments:
def my_view(request):
    ...
    return redirect("some-view-name", foo="bar")

# Using the redirect() function by passing an hardcoded URL:
def my_view(request):
    ...
    return redirect("/some/url/")
    # This also works with full URLs:
    # return redirect("https://example.com/")
```

By default, `redirect()` returns a temporary redirect.  
All of the above forms accept a permanent argument; if set to `True` a permanent redirect will be returned:
```python
def my_view(request):
    ...
    obj = MyModel.objects.get(...)
    return redirect(obj, permanent=True)
```

### View: Further reading
- [Django Views - The Right Way](https://spookylukey.github.io/django-views-the-right-way/index.html)
- [Classy Class-Based Views](https://ccbv.co.uk/)

## Routing
[Django Documentation: django.urls functions for use in URLconfs](https://docs.djangoproject.com/en/stable/ref/urls/)

- **path()**: Returns an element for inclusion in urlpatterns

```python
from django.urls import include, path

urlpatterns = [
    path("index/", views.index, name="main-view"),
    path("bio/<username>/", views.bio, name="bio"),
    path("articles/<slug:title>/", views.article, name="article-detail"),
    path("articles/<slug:title>/<int:section>/", views.section, name="article-section"),
    path("blog/", include("blog.urls")),
    ...,
]
```

- **re_path()**: Returns eturns an element for inclusion in urlpatterns.    
The route argument should be a string or `gettext_lazy()` that contains a regular expression compatible with Python’s re module.

```python
from django.urls import include, re_path

urlpatterns = [
    re_path(r"^index/$", views.index, name="index"),
    re_path(r"^bio/(?P<username>\w+)/$", views.bio, name="bio"),
    re_path(r"^blog/", include("blog.urls")),
    ...,
]
```

- **include()**: A function that takes a full Python import path to another URLconf module that should be “included” in this place.

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
	path("/admin/", admin.site.urls),
	path("books/", include ("books.urls")),
]
```

- **static()**: Helper function to return a URL pattern for serving files in debug mode.

```
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... the rest of your URLconf goes here ...
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## Authentication
### Authentication views and URLs
[Django Documentation: Using the views](https://docs.djangoproject.com/en/stable/topics/auth/default/#module-django.contrib.auth.views)

Add Django site authentication urls (for login, logout, password management):
```python
# config/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("accounts/", include("django.contrib.auth.urls")),
]
```

Urls provided by the auth app:
```
accounts/login/ [name='login']
accounts/logout/ [name='logout']
accounts/password_change/ [name='password_change']
accounts/password_change/done/ [name='password_change_done']
accounts/password_reset/ [name='password_reset']
accounts/password_reset/done/ [name='password_reset_done']
accounts/reset/<uidb64>/<token>/ [name='password_reset_confirm']
accounts/reset/done/ [name='password_reset_complete']
```

Updating `settings.py` with `LOGIN_REDIRECT_URL` and `LOGOUT_REDIRECT_URL`  
```
# config/urls.py
    ...
    path("", TemplateView.as_view(template_name="home.html"), name="home"),
    ...

# config/settings.py
LOGIN_REDIRECT_URL = "home"
LOGOUT_REDIRECT_URL = "home"
```

### Signup  
To create a sign up page we will need to make our own view and url.  
```python
python manage.py startapp accounts
```

```
# config/settings.py
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "accounts",
]
```
Then add a project-level url for the accounts app **above** our included Django auth app.
```
# django_project/urls.py
from django.contrib import admin
from django.urls import path, include
from django.views.generic.base import TemplateView

urlpatterns = [
    path("admin/", admin.site.urls),
    path("accounts/", include("accounts.urls")),  # new
    path("accounts/", include("django.contrib.auth.urls")),
    path("", TemplateView.as_view(template_name="home.html"), name="home"),
]
```
The *views* file:  
```
# accounts/views.py
from django.contrib.auth.forms import UserCreationForm
from django.urls import reverse_lazy
from django.views import generic


class SignUpView(generic.CreateView):
    form_class = UserCreationForm
    success_url = reverse_lazy("login")
    template_name = "registration/signup.html"
```
From [LearnDjango](https://learndjango.com/tutorials/django-signup-tutorial):
>We're subclassing the generic class-based view `CreateView` in our SignUp class. We specify the use of the built-in `UserCreationForm` and the not-yet-created template at `signup.html`. And we use `reverse_lazy` to redirect the user to the login page upon successful registration.

Create a new *urls* file in the *accounts* app.  
```
# accounts/urls.py
from django.urls import path

from .views import SignUpView


urlpatterns = [
    path("signup/", SignUpView.as_view(), name="signup"),
]
```

Then, create a new template templates/registration/signup.html  
```
<!-- templates/registration/signup.html -->
{% extends "base.html" %}

{% block title %}Sign Up{% endblock %}

{% block content %}
  <h2>Sign up</h2>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Sign Up</button>
  </form>
{% endblock %}
```

### Password reset
For development purposes Django let us store emails either in the console or as a file.

- Console backend:  
```
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```
- File backend:  
```
EMAIL_BACKEND = 'django.core.mail.backends.filebased.EmailBackend'
EMAIL_FILE_PATH = '/tmp/app-messages' # change this to a proper location
```

For production, see [Sending Email](#sending-email)

### OAuth
The [Django OAuth Toolkit](https://github.com/evonove/django-oauth-toolkit) package provides OAuth 2.0 support and uses [OAuthLib](https://github.com/idan/oauthlib).

### Authentication: Further reading
- [Django Documentation: User authentication in Django](https://docs.djangoproject.com/en/stable/topics/auth/)
- [LearnDjango: Django Login and Logout Tutorial](https://learndjango.com/tutorials/django-login-and-logout-tutorial)

## Custom Permissions
[Django Doc: Custom permissions](https://docs.djangoproject.com/en/stable/topics/auth/customizing/#custom-permissions)

Adding custom permissions to a Django model:
```python
from django.db import models

class Task(models.Model):
    title = models.CharField(max_length=70)
    body = models.TextField()
    is_opened = models.Boolean(default=False)

    class Meta:
        permissions = [
            ("set_task_status", "Can change the status of tasks")
        ]
```
the following checks if a user may close tasks:
```
user.has_perm("app.close_task")
```

**You still have to enforce it in the views**:

For **function-based views**, use the permission_required decorator:
```python
from django.contrib.auth.decorators import permission_required

@permission_required("book.view_book")
def book_list_view(request):
    return HttpResponse()
```

For **class-based views**, use the [PermissionRequiredMixin](https://docs.djangoproject.com/en/stable/topics/auth/default/#django.contrib.auth.mixins.PermissionRequiredMixin):
```python
from django.contrib.auth.mixins import PermissionRequiredMixin
from django.views.generic import ListView

from .models import Book

class BookListView(PermissionRequiredMixin, ListView):
    permission_required = "book.view_book"
    template_name = "books/book_list.html"
    model = Book
```
**permission_required** can be either a single permission or an iterable of permissions.
If using an iterable, the user must possess all the permissions in order to access the view.
```python
from django.contrib.auth.mixins import PermissionRequiredMixin
from django.views.generic import ListView

from .models import Book

class BookListView(PermissionRequiredMixin, ListView):
    permission_required = ("book.view_book", "book.add_book")
    template_name = "books/book_list.html"
    model = Book
```

**Checking permission in templates**:  
Using [perms](https://docs.djangoproject.com/en/stable/topics/auth/default/#permissions):
```
{% if perms.blog.view_post %}
  {# Your content here #}
{% endif %}
```
## Middleware
[Django Documentation: Middleware](https://docs.djangoproject.com/en/stable/ref/middleware/#middleware-ordering)

### Custom Middleware

```python
# my_app/custom_middlware.py

import time

class CustomMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        start_time = time.time()
        response = self.get_response(request)
        end_time = time.time()

        time_taken = end_time - start_time
        response['Time-Taken'] = str(time_taken)

        return response
```

Adding the custom middleware to our Django project:

```
MIDDLEWARE = [
    # ...
    'my_app.middleware.custom_middleware.CustomMiddleware',
    # ...
]
```

**Middleware ordering**  
While processing request object middlware works from top to bottom and while processing response object middleware works from bottom to top.  
[Django Documentation: Middleware ordering](https://docs.djangoproject.com/en/stable/ref/middleware/#middleware-ordering)

## Form and Form Validation
### Form
[Django Documentation: Creating forms from models](https://docs.djangoproject.com/en/stable/topics/forms/modelforms/#creating-forms-from-models)

ModelForm
```python
from django.forms import ModelForm
from myapp.models import Article

class ArticleForm(ModelForm):
     class Meta:
         model = Article
         fields = '__all__'
```

The view:
```python
#my_app/views.py

from .forms import ArticleForm

def article_create(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save()
            return redirect('article-detail', article.id))

    else:
        form = ArticleForm()

    return render(request,
            'listings/article_create.html',
            {'form': form})
```

### Selecting the fields to use
- Set the fields attribute to the special value `'__all__'` to indicate that all fields in the model should be used.

```python
from django.forms import ModelForm


class ArticleForm(ModelForm):
    class Meta:
        model = Article
        fields = "__all__"
```

- Set the exclude attribute of the ModelForm’s inner Meta class to a list of fields to be excluded from the form.

```python
class PartialAuthorForm(ModelForm):
    class Meta:
        model = Article
        exclude = ["headline"]
```

### Form template

```html
<form action="" method="POST">
    {% csrf_token %}
    {{ form }}
    <input type="submit" name="save" value="Save">
    <input type="submit" name="preview" value="Preview">
</form>
```

### Custom form field validators

```python
# my_app/validators.py
from django.core.exceptions import ValidationError


def validate_proton_mail(value):
    """Raise a ValidationError if the value doesn't contains @proton.me.
    """
    if "@proton.me" in value:
        return value
    else:
        raise ValidationError("This field accepts mail id of Proton Mail only")
```

Adding `validate_hello` in our form:

```python
# my_app/forms.py

from django import forms

from .models import MyModel
from .validators import validate_proton_mail

class MyModelForm(forms.ModelForm):

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['example_mail'].validators.append(validate_proton_mail)

    class Meta:
        model = MyModel
        fields = '__all__'

```

### clean()
Performing validation on more than one field at a time.

```python
# my_app/forms.py
from django import forms

from .models import MyModel


class MyForm(forms.ModelForm):

    class Meta:
        model = MyModel
        fields = '__all__'

    def clean(self):
        cleaned_data = super().clean()
        slug = cleaned_data.get('slug', '')
        title = cleaned_data.get('title', '')

        # slug and title should be same example
        if slug != title.lower():
            msg = "slug and title should be same"
            raise forms.ValidationError(msg)
        return cleaned_data
```

### clean_field_name()
Performing validation on a specific field.

```python
from django import forms

from .models import Product
from .validators import validate_amazing


class ProductForm(forms.ModelForm):

    class Meta:
        model = Product
        fields = '__all__'


    def clean_quantity(self):
        quantity = self.cleaned_data['quantity']
        if quantity > 100:
            msg = 'Quantity should be less than 100'
            raise forms.ValidationError(msg)
        return quantity
```

## Template
[Django Documentation: The Django template language](https://docs.djangoproject.com/en/stable/ref/templates/language/#the-django-template-language)
### Template inheritance and inclusion

- Inheritance

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <link rel="stylesheet" href="style.css">
    <title>{% block title %}My amazing site{% endblock %}</title>
</head>

<body>
    <div id="sidebar">
        {% block sidebar %}
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/blog/">Blog</a></li>
        </ul>
        {% endblock %}
    </div>

    <div id="content">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

```html
<!-- templates/home.html -->
{% extends "base.html" %}

{% block title %}My amazing blog{% endblock %}

{% block content %}
{% for entry in blog_entries %}
    <h2>{{ entry.title }}</h2>
    <p>{{ entry.body }}</p>
{% endfor %}
{% endblock %}
```

- Inclusion

```
{% include 'header.html' %}
```

### Common template tags  
- **static**

```
{% load static %}
{% static 'css/main.css' %}
```

- **url** passing positional arguments

```
{% url 'some-url-name' v1 v2 %}
```

[Django Documentation: url](https://docs.djangoproject.com/en/stable/ref/templates/builtins/#url)

- **block** (Defines a block that can be overridden by child templates)

```html
<div id="content">
        {% block content %}{% endblock %}
    </div>
```

A child template might look like this:

```html
{% extends "base.html" %}

{% block title %}My amazing blog{% endblock %}

{% block content %}
{% for entry in blog_entries %}
    <h2>{{ entry.title }}</h2>
    <p>{{ entry.body }}</p>
{% endfor %}
{% endblock %}
```

- **for**
 
```html
<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% endfor %}
</ul>
```

- **if, elif, else**

```html
{% if athlete_list %}
    Number of athletes: {{ athlete_list|length }}
{% elif athlete_in_locker_room_list %}
    Athletes should be out of the locker room soon!
{% else %}
    No athletes.
{% endif %}
```

- **now** (Outputs the current date and/or time.)

```
{% now "SHORT_DATETIME_FORMAT" %}
```

- **Current path**

```
{{ request.path }}
```

- **Dates and Times**

```html
<p>Copyright 2005-{% now "Y" %}</p>
```
- **Comments**

```html
{% comment "Optional note" %}
    <p>Commented out text with {{ create_date|date:"c" }}</p>
{% endcomment %}
```

Note that single lines of text can be commented out using `{#` and `#}`:

```
{# This is a comment. #}
```

- **Special Characters**

```
{% autoescape off %}
    {{ content }}
{% endautoescape %}
```

## Sending Email
[Django Documentation: Sending email](https://docs.djangoproject.com/en/stable/topics/email/#module-django.core.mail)

Quick example:

```python
from django.core.mail import send_mail

send_mail(
    "Subject here",
    "Here is the message.",
    "from@example.com",
    ["to@example.com"],
    fail_silently=False,
)
```

**Email backend**

Development:

```
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

Production:

```
config/settings.py
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.yourserver.com'
EMAIL_USE_TLS = False
EMAIL_PORT = 465
EMAIL_USE_SSL = True
EMAIL_HOST_USER = 'your@djangoapp.com'
EMAIL_HOST_PASSWORD = 'your password'
```

## Performance
### django-debug-toolbar
[Django Debug Toolbar Documentation](https://django-debug-toolbar.readthedocs.io/en/latest/)

Install:  
```python -m pip install django-debug-toolbar```

`settings.py`
```
INSTALLED_APPS = [
    # ...
    "debug_toolbar",
    # ...
]
```

`urls.py`
```
from django.urls import include, path

urlpatterns = [
    # ...
    path("__debug__/", include("debug_toolbar.urls")),
]
```


### select_related and prefetch_related
Django provides two QuerySet methods that can turn the N queries back into one query, solving the performance issue.

These two methods are:

**select_related**  
[Django Documentation: select_related](https://docs.djangoproject.com/en/stable/ref/models/querysets/#select-related)

select_related returns a QuerySet that will “follow” foreign-key relationships (either One-to-One
or One-to-Many), selecting additional related-object data when it executes its query.

```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=50)

class Book(models.Model):
    title = models.CharField(max_length=50)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

# With select_related
authors = Author.objects.all().select_related('book')
for author in authors:
    books = author.book_set.all()
```

**prefetch_related**  
[Django Documentation: prefetch_related](https://docs.djangoproject.com/en/stable/ref/models/querysets/#prefetch-related)

`prefetch_related` performs a separate lookup for each relationship and “joins” them together
with Python, not SQL.  
This allows it to prefetch many-to-many and many-to-one objects, which
cannot be done using select_related
```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=50)

class Book(models.Model):
    title = models.CharField(max_length=50)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

# With prefetch_related
authors = Author.objects.all().prefetch_related('book')
for author in authors:
    books = author.book_set.all()
```

### Indexes
[Django Documentation: Model index reference](https://docs.djangoproject.com/en/stable/ref/models/indexes/#module-django.db.models.indexes)

If a particular field is consistently utilized, accounting for around 10-25% of all queries, it is a prime candidate
to be indexed.  
The downside is that indexes require additional space on a disk so they must be used with care.

```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=50)
    
    class Meta:
    		indexes = [models.Index(fields=["name"])]
```

### Caching
#### Redis
[Django Documentation: Redis](https://docs.djangoproject.com/en/stable/topics/cache/#redis)

1. Setting up a [Redis](https://redis.io/) server locally or on a remote machine.
2. Installing [redis-py](https://pypi.org/project/redis/). Installing [hiredis-py](https://pypi.org/project/hiredis/) is also recommended.
3. Set `BACKEND` to `django.core.cache.backends.redis.RedisCache`.
4. Set `LOCATION` to the URL pointing to your Redis instance, using the appropriate scheme.
See the redis-py docs for [details on the available schemes](https://redis-py.readthedocs.io/en/stable/connections.html#redis.connection.ConnectionPool.from_url).
For example, if Redis is running on localhost (127.0.0.1) port 6379:

```
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.redis.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379",
    }
}
```

In order to supply a username and password, add them in the `LOCATION` along with the `URL`:

 ```
 CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.redis.RedisCache",
        "LOCATION": [
            "redis://127.0.0.1:6379",  # leader
            "redis://127.0.0.1:6378",  # read-replica 1
            "redis://127.0.0.1:6377",  # read-replica 2
        ],
    }
}
 ```

#### Database caching
[Django Documentation: Database caching](https://docs.djangoproject.com/en/stable/topics/cache/#database-caching)

Django can store its cached data in your database.
This works best if you’ve got a fast, well-indexed database server.
In this example, the cache table’s name is `my_cache_table`:

```
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.db.DatabaseCache",
        "LOCATION": "my_cache_table",
    }
}
```

Creating the cache table:

```
python manage.py createcachetable
```

#### per-view cache
[Django Documentation: The per-view cache](https://docs.djangoproject.com/en/stable/topics/cache/#the-per-view-cache)

In Django, the `cache_page` decorator is used to cache the output of a view function.
It takes a single argument, timeout, which specifies the duration in seconds for which the output should be cached.

```python
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # Cache the page for 15 minutes
def my_view(request):
    # View logic ...
```

Specifying per-view cache in the `URLconf`:  
You can do so by wrapping the view function with cache_page when you refer to it in the URLconf.

```python
from django.views.decorators.cache import cache_page

urlpatterns = [
    path("foo/<int:code>/", cache_page(60 * 15)(my_view)),
]
```

#### per-site cache
[Django Documentation: per-site cache](https://docs.djangoproject.com/en/stable/topics/cache/#the-per-site-cache)

```
# config/settings.py

MIDDLEWARE = [
    'django.middleware.cache.UpdateCacheMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware',
]
```

**The order of the middleware is important**  
`UpdateCacheMiddleware` must come before `FetchFromCacheMiddleware`.

#### template fragment caching
[Django Documentation: Template fragment caching](https://docs.djangoproject.com/en/stable/topics/cache/#template-fragment-caching)

```html
{% load cache %}

{% cache 500 book_list %}
  <ul>
    {% for book in books %}
      <li>{{ book.title }}</li>
    {% endfor %}
  </ul>
{% endcache %}
```

The `cache` template tag expects a cache timeout in second with the name of the cache fragment `book_list`

## Security
[Django Documentation: Deployment checklist](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/)

### Admin Hardening
Changing the URL path

```python
# config/urls.py

from django.contrib import admin
from django.urls import path

urlpatterns = [
    path("another_admin_path/", admin.site.urls),
]

```

### Cross site request forgery (CSRF) protection
Django's CSRF protection is turned on by default. You should always use the `{% csrf_token %}` template tag in your forms and use `POST` for requests that might change or add data to the database.

### Enforcing SSL HTTPS
- [SECURE_PROXY_SSL_HEADER](https://docs.djangoproject.com/en/stable/ref/settings/#std:setting-SECURE_PROXY_SSL_HEADER): can be used to check whether content is secure, even if it is incoming from a non-HTTP proxy.
- HSTS may either be configured with [SECURE_HSTS_SECONDS](https://docs.djangoproject.com/en/stable/ref/settings/#std:setting-SECURE_HSTS_SECONDS) and [SECURE_HSTS_INCLUDE_SUBDOMAINS](https://docs.djangoproject.com/en/stable/ref/settings/#std:setting-SECURE_HSTS_INCLUDE_SUBDOMAINS) or on the Web server.
- To ensure that cookies are only ever sent over HTTPS, set [SESSION_COOKIE_SECURE](https://docs.djangoproject.com/en/stable/ref/settings/#std:setting-SESSION_COOKIE_SECURE) and [SECURE_HSTS_SECONDS](https://docs.djangoproject.com/en/stable/ref/settings/#std:setting-SECURE_HSTS_INCLUDE_SUBDOMAINS) to `True`

### ALLOWED_HOSTS
[Django Documentation: ALLOWED_HOSTS](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-ALLOWED_HOSTS)
Use `ALLOWED_HOSTS` to only accept requests from trusted hosts.

## Further Reading
- [Official Django Documentation](https://www.djangoproject.com/)
- [Django source code](https://github.com/django/django)
- [Official Django forum](https://forum.djangoproject.com/)
- Will Vincent:
	- [LearnDjango](https://learndjango.com/)
	- Django for [Beginners](https://djangoforbeginners.com/), [Professionals](https://djangoforprofessionals.com/), [APIs](https://djangoforapis.com/)
	- [Awesome Django](https://github.com/wsvincent/awesome-django) (with [Jeff Triplett](https://github.com/jefftriplett))
- Adam Johnson:  [Blog](https://adamj.eu/tech/), [Boost Your Django DX](https://adamchainz.gumroad.com/l/byddx)
- [mdn web docs: Django Web Framework](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django)
- [Django Styleguide](https://github.com/HackSoftware/Django-Styleguide)
- [Simple is better than complex](https://simpleisbetterthancomplex.com/)
- [Two scoops of Django 3.x](https://www.feldroy.com/books/two-scoops-of-django-3-x)
