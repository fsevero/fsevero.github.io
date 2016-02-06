---
layout: post
title: Configurando o projeto Django
---
"Tutorial" explicando como criar e organizar o projeto [Django](https://www.djangoproject.com/).

**OBSERVAÇÃO IMPORTANTE:**
Todas as informações nesse post foram retiradas através das interações ocorridas no curso [Welcome to the Django](http://welcometothedjango.com.br/),
ministrado pelo [Henrique Bastos](http://henriquebastos.net/).

Para gerarmos o código Django dentro do projeto, devemos iniciar um “projeto Django”, através do seguinte código:

```bash
$ django-admin startproject project_name .
```

Observe que o “ponto” no final do comando é proposital,
e serve para criar a estrutura do projeto na pasta atual (não criar um subdiretório).
Este comando irá criar um arquivo `manage.py` e um diretório `project_name`

O arquivo `manage.py`, como o próprio nome já demonstra,
é um “gerenciador” que contém uma lista de comandos úteis e importantes para o desenvolvimento,
como pode ser visto aqui (utilizando `python manage.py` ou `manage` caso utilizado o alias do post anterior):

```bash
Type 'manage.py help <subcommand>' for help on a specific subcommand.
 
Available subcommands:
 
[auth]
    changepassword
    createsuperuser
 
[django]
    check
    compilemessages
    createcachetable
    dbshell
    diffsettings
    dumpdata
    flush
    inspectdb
    loaddata
    makemessages
    makemigrations
    migrate
    sendtestemail
    shell
    showmigrations
    sqlflush
    sqlmigrate
    sqlsequencereset
    squashmigrations
    startapp
    startproject
    test
    testserver
 
[sessions]
    clearsessions
 
[staticfiles]
    collectstatic
    findstatic
    runserver
```

Este arquivo contém o gerenciamento do servidor de desenvolvimento, gerenciamento de banco de dados, “templates” de apps, etc.

Neste momento, você já pode ver o django funcionando rodando o seguinte comando
e visualizando o endereço correspondente (`http://localhost:8000` por padrão):

```bash
$ python manage.py runserver
```

Durante esse comando, serão apresentados alguns avisos (em vermelho) pedindo a utilização do comando `python manage.py migrate`.
Este comando não precisa ser executado no momento, e serve para aplicar todas as modificações necessárias no banco de dados.
Porém, sequer temos um banco de dados configurado ainda! =D

Dentro da pasta `project_name` teremos os arquivos:

* **\_\_init\_\_.py:** Define que a pasta `project_name` é um módulo python
* **settings.py:** Configurações do seu projeto (padrão)
* **urls.py:** ponto de entrada (padrão) da validação das rotas (urls)
* **wsgi.py:** “[Web Server Gateway Interface](https://pt.wikipedia.org/wiki/Web_Server_Gateway_Interface)“, serve como interface entre o servidor web (nginx, por exemplo) e a aplicação (o projeto django)

No arquivo `wsgi.py` a única alteração (por enquanto) que vamos fazer,
é dizer para a aplicação “ignorar” qualquer requisição por arquivos estáticos,
e simplesmente entregá-los. Para isso, deve ter sido instalado o `dj-static` (`pip install dj-static`).
Seu arquivo ficará assim:

```python
import os
from dj_static import Cling
from django.core.wsgi import get_wsgi_application
 
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "project_name.settings")
 
application = Cling(get_wsgi_application())
```

As modificações feitas foram: importar o `Cling` e “circundar” o `get_wsgi_application()` com o mesmo.
Desta forma, o `Cling` irá validar se a requisição é um arquivo estático ou não.
Caso for, vai entregá-lo; caso contrário, encaminha a requisição para a aplicação.

No arquivo `settings.py`, precisamos configurar algumas coisas e, importante, manter um único arquivo de configuração!
Algumas abordagens preveem “quebrar” o settings em “base”, “dev”, “test”, “production”.
Essa abordagem pode causar problemas para manutenção, uma vez que cada alteração precisa ser executada e
avaliada em todos os arquivos, para evitar que sejam enviados erros ou inconsistências.

Para evitar a quebra do settings, podemos fazer uso do o `python-decouple`.
Ele ira configurar valores padrões para algumas configurações
e permitir que estes valores sejam alterados por variáveis de ambiente ou um arquivo de configuração local
(da instância, não do projeto).

Alguns exemplos de como o `Decouple` funciona a partir de algumas modificações que precisamos fazer no `settings.py`:

```python
from decouple import config, Csv
from dj_database_url import parse as dburl
 
# Secret não tem valor padrão, e deve ser setado nas variáveis da instância.
SECRET_KEY = config('SECRET_KEY')
 
# DEBUG é, por padrão, falso. Mas em desenvolvimento quero ele como true
DEBUG = config('DEBUG', default=False, cast=bool)
 
# Por padrão, deixamos acessar a aplicação somente o localhost em desenvolvimento
# mas precisamos setar, em produção, o HOST de produção.
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default=[], cast=Csv())
 
# Padrão: banco de dados sqlite3 (arquivo junto do projeto)
# em produção, usaremos a "url" do banco de dados
default_dburl = 'sqlite:///' + os.path.join(BASE_DIR, 'db.sqlite3')
DATABASES = {
    'default': config('DATABASE_URL', default=default_dburl, cast=dburl),
}
 
# OUTRAS MODIFICAÇÕES
LANGUAGE_CODE = 'pt-br'
TIME_ZONE = 'America/Sao_Paulo'
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
MEDIA_URL = '/static/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

Observação: Estamos tratando os arquivos estáticos (`static`) e uploads (`media`) como “parte do projeto”.
Em um projeto maior, deveríamos “separar” os arquivos e utilizar um servidor dedicado
([CDN](https://pt.wikipedia.org/wiki/Content_Delivery_Network)) para entregá-los,
mantendo lógica da aplicação em um servidor (sem afetar performance afetada),
e arquivos estáticos e medias em um servidor especializado em entregar conteúdo estático.
Irei abordar esse assunto em um outro momento.

Maiores informações sobre as possíveis configurações do Django no `settings.py` na
[Documentação Oficial](https://docs.djangoproject.com/en/1.9/topics/settings/)
e [Neste Post do Patrick Mazulo](http://blog.dunderlabs.com/serie-django-settings.html)

Após as configurações básicas, vamos criar as configurações da nossa instância de desenvolvimento.
Dentro da pasta `project_folder` (junto ao `manage.py`), crie um arquivo `.env` com o seguinte conteúdo:

```
SECRET_KEY=ThisSecretIsNotSoSecret
DEBUG=True
ALLOWED_HOSTS=127.0.0.1, .localhost
```
