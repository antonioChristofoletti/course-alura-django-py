# course-alura-django-py

Learning about the Django Python Web Framework

# Introduction

Interesting doc: `https://docs.djangoproject.com/pt-br/2.2/misc/design-philosophies/`

Django is a framework used in order to create webserver (website/web api) using python application/code as backend and 
providing the pages.

Django takes carry all the major points: good syntax, security, ORM, MVT architecture (MVC), admin interface, 
good dev community and so on.

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

-> Firstly It is necessary define the place in which these files are going to stay:

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

# Interactive Shell

Django allows interactive terminal shell using the command: `pytho manage.py shell` (ctrl+d to exit)

For instance, It's useful to check data in the database:

```
>>> from galeria.models import Fotografia
>>> foto = Fotografia(nome="Nebulosa de Carina", legenda="webbtelescope.org / NASA / James Webb", foto="carina-nebula.png")
>>> print(foto)
Fotografia [nome=Nebulosa de Carina]
>>> foto.save()
>>> Fotografia.objects.all()
<QuerySet [<Fotografia: Fotografia [nome=Nebulosa de Carina]>]>
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

3- Configure the table in which this information is going to be saved:

```
class Receita(models.Model):
    foto_receita = models.ImageField(upload_to='fotos/%d/%m/%Y/', blank=True)
```

4- Then just pass the url value in the html page. Example:

```
<img src="{{ receita.foto_receita.url }}" alt="">
```

# Django Forms

Django module that allows to define and control html forms using python notation.

Advantages: simple, use the DRY principles and increase development speed.

Disadvantages: Can get tricky to create custom behaviors.

Example:

```
class LoginForms(forms.Form):
    nome_login = forms.CharField(
        label="Nome de Login",
        required=True,
        max_length=100,
        widget=forms.TextInput(
            attrs={
                "class": "form-control",
                "placeholder": "Ex.: João Silva"
            }
        )
    )
    senha = forms.CharField(
        label="Senha",
        required=True,
        max_length=70,
        widget=forms.PasswordInput(
            attrs={
                "class": "form-control",
                "placeholder": "Digite sua senha"
            }
        )
    )
```

Good practice keep these configs in a `form.py` for each app. There are many different configuration that can be applied.

To use them in the template is pretty straight forward:

Obs.: Important use `{% csrf_token %}` to avoid CSRF attacks.

```
<form action="" method="">
    {% csrf_token %}
    <div class="row">
        {% for field in form.visible_fields %}
            <div class="col-12 col-lg-12" style="margin-bottom: 10px;">
                <label for="{{ field.id_for_label }}" style="color:#D9D9D9; margin-bottom: 5px;">{{field.label}}</label>
                {{ field }}
            </div>
        {% endfor %}
    </div>
    <div>
        <button class="btn btn-success col-12" style="padding: top 5px;" type="submit">Logar</button>
    </div>
</form>
```

Inside the form class It is possible define validations. For examplo:

```
def clean_name(self):
    name = self.cleaned_data.get("name")

    if not name:
        return

    name = name.strip()

    if " " in name:
        raise forms.ValidationError("Name cant have spaces")
    else:
        return nome
```

These validations can be shown in the client adding this config inside the form (You can change the message style):

```
{% for error in field.errors %}
    <div class="alert alert-danger">
        {{ error }}
    </div>
{% endfor %}
```

Django allows to add tags to different types of messages to help identifying and style messages.
Add this code in the `settings.py`

```
from django.contrib.messages import constants as messages
MESSAGE_TAGS = {
    messages.ERROR: 'danger',
    messages.SUCCESS: 'success',
}
```

It is possible to link a form with a model, reducing the quantity of configuration. Example:

```
class FotografiaForms(forms.ModelForm):
    class Meta:
        model = Fotografia
        exclude = ['publicada', ]
        labels = {
            'descricao': 'Descrição',
            'data_fotografia': 'Data de Registro',
            'usuario': 'Usuário'
        }

        widgets = {
            'nome': forms.TextInput(attrs={'class': 'form-control'}),
            'legenda': forms.TextInput(attrs={'class': 'form-control'}),
            'categoria': forms.Select(attrs={'class': 'form-control'}),
            'descricao': forms.Textarea(attrs={'class': 'form-control'}),
            'foto': forms.FileInput(attrs={'class': 'form-control'}),
            'data_fotografia': forms.DateInput(
                format='%d/%m/%Y',
                attrs={
                    'class': 'form-control',
                    'type': 'date'
                }
            ),
            'usuario': forms.Select(attrs={'class': 'form-control'}),
        }
```

The process of handling the form is also simplified. Example:

```
if not request.user.is_authenticated:
    messages.error(request, 'Usuário não logado')
    return redirect('login')

form = FotografiaForms
if request.method == 'POST':
    form = FotografiaForms(request.POST, request.FILES)
    if form.is_valid():
        form.save()
        messages.success(request, 'Nova fotografia cadastrada')
        return redirect('index')

return render(request, 'galeria/nova_imagem.html', {'form': FotografiaForms()})
```

# Handling requests

Pretty simples, It is not necessary define the parameters in the urls.py file.

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

Handling create new user and login is quite simple too:

```
def cadastro(request):
    if request.method != "POST":
        form = CadastroForms()
        return render(request, 'usuarios/cadastro.html', {"form": form})

    form = CadastroForms(request.POST)

    if not form.is_valid():
        return redirect('cadastro')

    if form["senha_1"].value() != form["senha_2"].value():
        messages.error(request, f"Senhas não são iguais")
        return redirect('cadastro')

    nome = form["nome_cadastro"].value()
    email = form["email"].value()
    senha = form["senha_1"].value()

    if User.objects.filter(username=nome).exists():
        messages.error(request, f"Usuário já existente")
        return redirect('cadastro')

    usuario = User.objects.create_user(username=nome, email=email, password=senha)
    usuario.save()
    messages.success(request, f"Cadastro efetuado com sucesso")
    return redirect('login')
```

Obs.: The messages are going to be sent as arguments to be rendered by jinja.

Example of how handle messages sent by backend to the client:

```
{% for message in messages %}
    <div class="alert alert-primary">
        <p id="messages">{{ message }}</p>
    </div>
{% endfor %}
```