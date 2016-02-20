---
layout: post
title: Django - Trabalhando com banco de dados legado
---

A alguns dias tenho vontade de replicar uma aplicação em PHP+Cake em Python+Django.
O problema dessa mudança é que não seria atômica, e eu não teria controle sobre o banco de dados.
Caso viesse a fazer este experimento, precisaria manter ambas as aplicações rodando simultâneamente.
Minha primeira preocupação: Como utilizar o Django em uma aplicação já existente, sem que o mesmo utilize as migrations.
Neste post contarei um pouco sobre minha primeira experiência.

Primeiramente contei com o apoio e a ajuda do 
[Régis](http://pythonclub.com.br/author/regis-da-silva.html) e do 
[Cuducos](http://cuducos.me/),
os quais forneceram informações e incentivo para pesquisar e escrever este post.

Para iniciar meu experimento, criei uma aplicação django "do zero":

```bash
$ mkdir django_experiment
$ cd django_experiment
$ mkvirtualenv django_experiment
$ pip install django mysqlclient
$ django-admin startproject django .
$ python manage.py startapp core
```

As configurações para rodar Django com o MySQL eu consegui no
[stackoverflow](http://stackoverflow.com/questions/19189813/setting-django-up-to-use-mysql).

Com essa estrutura básica pronta 
([Tenho tutoriais sobre como iniciar e configurar o projeto](http://fsevero.github.io/django-iniciando-projeto/)),
Podemos começar vinculandoa nossa app e o nosso projeto com o banco legado, através da configuração no `settings.py`:

```python
INSTALLED_APPS = (
  # ...
  'core',
)

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', 
        'NAME': 'DB_NAME',
        'USER': 'DB_USER',
        'PASSWORD': 'DB_PASSWORD',
        'HOST': 'localhost',   # Or an IP Address that your DB is hosted on
        'PORT': '3306',
    }
}
```

Com acesso ao banco de dados, podemos fazer com que o Django "inspecione" o mesmo e gere os nossos modelos:

```bash
$ python manage.py inspectdb
```

Este comando irá imprimir no terminal todos os registros que o Django econtrou e mapeou na forma de classes.
Para salvar estes dados, mude o comando para:

```bash
$ python manage.py inspectdb > models_inspected.py
```

Este comando irá redirecionar a saída do terminal para um arquivo chamado `models_inspected.py` no diretório atual.
Eu preferí fazer dessa forma, pois meu banco de dados possui quase 500 tabelas!
Criando um novo arquivo (e não alterando diretamente o `core/models.py`)
eu tenho uma melhor clareza e posso ir "copiando e ajustando" aos poucos meus modelos.

Pronto. Você tem uma estrutura inicial para começar a transfomar com o código gerado automaticamnente.
Dependendo da organização do seu banco de dados, você pode sobreescrever o models.py diretamente e começar a utilizá-lo.

Alguns pontos que levantei:

* Como o código é gerado automaticamente, a qualidade dele é diretamente proporcional à qualidade do seu banco de dados.
* Você precisará de ajustes, mas é muito mais fácil aplicar algumas modificações que construir tudo do zero.
* o atributo `managed` setado como `False` diz que esta classe não será gerenciada pelas migrações,
portanto você não precisa rodar os comandos `makemigrations`e `migrate`. (Dica do Cuducos!)
* Caso queira ir transcrevendo seu código aos poucos, você pode versionar o arquivo em uma outra pasta como,
por exemplo, `legacy_data/models.py`.
* Você precisará ajustar os métodos `__str__` das suas classes.
* ...

Para verificar que tudo ficou correto, fiz os seguintes testes:

```bash
$ python manage.py shell
```

Abrindo o interpretador:

```python
>>> from core.models import User
>>> User.objects.all()
# lista de usuários
>>> User.objects.first()
# Primeiro usuário
>>> from django.core import serializers
>>> serializers.serialize('json', User.objects.all())
# Lista de usuários em formato JSON
```

Tudo funcionando, com o gerenciamento do banco de dados realizado pelo PHP e acessando os dados pelo Django!

Próximo passo: Utilizar o 
[tutorial do Régis sobre Django Rest Framework](http://pythonclub.com.br/django-rest-framework-quickstart.html)
e iniciar o teste da API.
