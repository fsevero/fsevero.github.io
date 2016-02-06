---
layout: post
title: Criando uma landing page com o Django
---
"Tutorial" explicando como organizar seus arquivos/apps e criar uma landing page simples com [Django](https://www.djangoproject.com/).

**OBSERVAÇÃO IMPORTANTE:**
Todas as informações nesse post foram retiradas através das interações ocorridas no curso [Welcome to the Django](http://welcometothedjango.com.br/),
ministrado pelo [Henrique Bastos](http://henriquebastos.net/).

Ok, temos nosso projeto com o código django já configurado. Agora, onde vai meu código?

Temos a seguinte estrutura de pastas:

```bash
.
├── manage.py
└── project_name
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

Para começarmos a programar algo no Django, precisamos incluir uma APP.
Segundo o [site do Django](https://docs.djangoproject.com/en/1.9/intro/tutorial01/#creating-the-polls-app):

> What’s the difference between a project and an app?
> An app is a Web application that does something – e.g., a Weblog system, a database of public records or a simple poll app.
> A project is a collection of configuration and apps for a particular website.
> A project can contain multiple apps. An app can be in multiple projects.

Ou seja, uma APP é onde seu código vive. Mas não pense seu sistema em APPs.
Comece seu projeto de forma simples, e vá extraindo código para APPs específicas conforme o código for crescendo.
Naturalmente o código irá começar a apresentar diferenças e especificidades,
“pedindo” naturalmente para ser extraído para uma outra APP.

Seguindo esse conselho, criamos sempre uma APP `core`! Ela terá todo núcleo do código do projeto.
Conforme o código for evoluindo, as funcionalidades do core começam a ser extraídas para novas APPs.
Neste `core` estarão configurações, templates, layouts, models, etc. Tudo que servirá como “base” para o projeto.

Então, para criar nossa primeira app:

```bash
$ cd project_name
$ manage startapp core
# ou python ../manage.py startapp core
```

A estrutura de pastas ficará assim:

```bash
.
├── manage.py
└── project_name
    ├── core
    │   ├── admin.py
    │   ├── apps.py
    │   ├── __init__.py
    │   ├── migrations
    │   │   └── __init__.py
    │   ├── models.py
    │   ├── tests.py
    │   └── views.py
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

Inicialmente, todas as apps serão organizadas dentro do projeto Django (pasta `project_name`).
Caso o número de apps cresça muito (e somente nesse caso), podemos criar uma pasta apps para conter todas elas.
Mas somente pensaremos nessa opção e como fazer para organizar quando isso for necessário.

Após gerar a app core, devemos “instalá-la” no projeto Django.
Para isso, vamos até o arquivo `settings.py` e ajustar a variável `INSTALLED_APPS`, acrescentando o seguinte código:

```python
INSTALLED_APPS = (
    # ...
    'project_name.core',
)
```

Agora, vamos criar uma `view`, que tem como responsabilidade receber um **request**,
**orquestrar** os dados, e devolver um **response**. No arquivo `core/views.py`, digite o seguinte código:

```python
from django.shortcuts import render

def home(request):
    context_dict = None
    return render(request, 'index.html', context_dict)
```

O `context_dict` não é importante aqui, deixei ele somente para mostrar que é possível passar dados para o template a ser renderizado.

Para que essa view seja renderizada corretamente, é necessário criar o arquivo `index.html`.
Ele deve ser criado em `core/templates/index.html`
(Fique livre para colocar o código HTML que você preferir, pode ser um texto simples também).

Para utilização dos arquivos estáticos (css, imagens, etc) nos templates:

```
{% raw %}
# Carrega as template tags
{% load static %}
 
# Adiciona um arquivo estático no template
{% static 'folder/file' %}
 
# Regex para ajustar caminhos "hard-coded"
(src|href)="((img|css|js).*?)" 
# PARA
$1="{% static '$2' %}"
{% endraw %}
```

Agora falta criarmos a rota expondo a view. Para isso, no arquivo `urls.py`, adicionar o seguinte código:

```python
from project_name.core.views import home
 
urlpatterns = [
    url(r'^$', home, name='home'),
    # ...
]
```

Caso o arquivo de urls comece a descer ou tenha muitas urls semelhantes de uma única app,
podemos quebrar esse arquivo, e criar, dentro da app `core` um arquivo `urls.py`. Mantendo assim:

```python
# urls.py do project_name
urlpatterns = [
    # ...
    url(r'^homepage/',
        include('project_name.core.urls',
        namespace='core')),
    # ...
]
 
# urls.py do core
from project_name.core.views import home
 
urlpatterns = [
    url(r'^$', home, name='home'),
]
```

Dessa forma, a view `home` estará exposta em `/homepage/`.

**Obs.:** A url no `include` pode muito bem ser especificado como "vazia" (`r'^'`). Desta forma, a url exposta para a view `home` seria apenas `/`
