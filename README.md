# course-alura-django-py

Learning about the Django Python Web Framework

# Introduction

Interesting doc: `https://docs.djangoproject.com/pt-br/2.2/misc/design-philosophies/`

Django is a framework used in order to create websites using python application/code as backend and providing the pages.

## Apps and Project

Django has the concept of apps and projects.

Apps = It is the application itself, the website, has pages, features, code, html, css, javascript etcetra...
Project = Collection of configuration that is going to be set for the sites.

An app can be in multiple projects, It is portable.

# Starting up a project

In a venv environment..

-> Dependency: `pip install Django`;
-> Creating a project: `django-admin startproject your_name_project .`
-> Start server: `python manage.py runserver`
-> Creating an app: `python manage.py startapp your_app_name`

# Static Files

-> Firstly It is necessary define the place in which this files is going to stay:

Define this in the file setting.py of the project:
```
# Static root is the absolute folder in which the django put the static files after execute collectstatic for example.
Django always manage this folder automatically
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

# Configuration used in order to refer to STATIC_ROT folder
STATIC_URL = '/static/'

# Define the places in which the FileSystemFinder is going to look if enable.
# Commands as collectstatic or findstatic is going to use this config in order to do their jobs. 
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'alurareceita/static')
]
```

The template file (htmls) need a special configuration for the django knows that imgs, css and others external files 
need to be provided by him.
Example:

```
    {% load static  %}

    <link rel="icon" href="{% static 'img/core-img/favicon.ico' %}">
    <link rel="stylesheet" href="{% static 'site.css' %}">
    <script src="{% static 'js/jquery/jquery-2.2.4.min.js' %}"></script>
```

# Reusing code page

## Including html in another page

Good for inserting piece of code in others page.

Example:

```{% include 'partials/menu.html' %}```

## Extending a page

Good for extending a default page structure for example.

Page being extended Example:
```
html code.....

{% block content%} {% endblock %}

html code....
```

This part of python code is where the other page is going to be allocated.

Page extending other Example:
```
{% extends 'base.html' %}
html code......
{% endblock %}
```

# Dynamic pages with data

In the python code in which the template is going to be called:

```
page_data = {
    'food_list': {
    1: 'Egg',
    2: 'Rice',
    3: 'Bean',
    4: 'Milk'
}
}

return render(request, 'index.html', page_data)
```

in the html file:
```
{% for _, food_name in food_list.items %}
    <!-- Single Best Receipe Area -->
    <div class="col-12 col-sm-6 col-lg-4">
        <div class="single-best-receipe-area mb-30">
            <img src="{% static 'img/bg-img/foto_receita.png' %}" alt="">
            <div class="receipe-content">
                <a href="receita.html">
                    <h5>{{ food_name }}</h5>
                </a>
            </div>
        </div>
    </div>
{% endfor %}
```

# Databases

## Configuring (POSTGRES)

Install libs:

```commandline
pip install psycopg2
pip install psycopg2-binary
```

In the settings.py of project folder change:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgressql',
        'NAME': 'alura_receita',
        'USER': 'admin',
        'PASSWORD': 'admin',
        'HOST': 'localhost'
    }
}
```

## Models - Managing Schema

In the app's models.py set the models that going to create/update the schemas in the database automatically:

```
class Receita(models.Model):
    pessoa = models.ForeignKey(Pessoa, on_delete=models.CASCADE)
    nome_receita = models.CharField(max_length=200)
    ingredientes = models.TextField()
    modo_preparo = models.TextField()
    tempo_preparo = models.IntegerField()
    rendimento = models.CharField(max_length=100)
    categoria = models.CharField(max_length=100)
    date_receita = models.DateTimeField(default=datetime.now(), blank=True)
    foto_receita = models.ImageField(upload_to='fotos/%d/%m/%Y/', blank=True)
    publicada = models.BooleanField(default=False)
```

Django does not apply automatically, It is necessary:

1- Cmd: `python manage.py makemigrations` (Create new migrations to apply in the database)
2- Cmd: `python manage.py migrate` (Synchronize the migrations and the database schema)

## Selecting table's content to feed pages

Pretty simple, return object following the model's attribute name.

Example in the view.py

```
from django.shortcuts import render
from .models import Receita

def index(request):
    receitas = Receita.objects.all()

    dados = {
        'receitas': receitas
    }

    return render(request, 'index.html', dados)

```

# Django Admin

## What is

Django model that works as a CRUD to manage the tables in the database mapped by the models.

Generally It is mapped in the url: "your_url/admin"

It is necessary creates a user to access: `python manage.py createsuperuser`

Only the models mapped in the apps's admin.py file can be managed in the Django Admin:

```
from django.contrib import admin
from .models import Receita

admin.site.register(Receita)
```

## Customizing Models

Django admin brings the mapped tables in a nice ui to manage their content, however, It is possible customize in order to
show specific cols in a defined order, links to edit the values, paginations and others features.

Example:

Obs.: Editing the App's admin.py
```
class ListandoReceitas(admin.ModelAdmin):
    list_display = ('id', 'nome_receita', 'categoria', 'tempo_preparo', 'publicada')
    list_display_links = ('id', 'nome_receita')
    search_fields = ('nome_receita',)
    list_filter = ('categoria',)
    list_editable = ('publicada',)
    list_per_page = 100

admin.site.register(Receita, ListandoReceitas)
```

# Dynamic URL Parameter

First of all, in the template It is going to be necessary to indicate which parameter is going to be transmited to the next page. Example:

```
<a href="{% url 'receita' receita.id %}">
```

receita.id is the parameter.

The View needs to be prepared to receive the parameter. Example:

```
def receita(request, receita_id):

    # Make a search in the database
    receita = get_object_or_404(Receita, pk=receita_id)

    receita_a_exibir = {
        'receita': receita
    }

    return render(request, 'receita.html', receita_a_exibir)
```

The url needs to be configured too:

```
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:receita_id>', views.receita, name='receita')
]
```

# Media

1- Add the configuration in the project's settings.py:
```
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
```

2- Allow the media urls be accessible outside in the project's settings.py:
```
urlpatterns = [
    path('', include('receitas.urls')),
    path('admin/', admin.site.urls),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

3- Configure the table in which these informations is going to be saved:

```
class Receita(models.Model):
    foto_receita = models.ImageField(upload_to='fotos/%d/%m/%Y/', blank=True)
```

4- Then just pass the url value in the html page. Example:

```
<img src="{{ receita.foto_receita.url }}" alt="">
```

# GET Parameters

Pretty simples, It is not necessary define the parameters in thr urls.py file.

Just need to get and use them in the views. Example:

```
 receitas = Receita.objects.order_by('-date_receita').filter(publicada=True)
    if 'buscar' in request.GET:
        nome_buscar = request.GET['buscar']
        if nome_buscar:
            receitas = receitas.filter(nome_receita__icontains=nome_buscar)

    dados = {
        'receitas': receitas
    }
```