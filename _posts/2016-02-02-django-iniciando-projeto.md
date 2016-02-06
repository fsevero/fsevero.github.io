---
layout: post
title: Iniciando um projeto com o Django
---
"Tutorial" explicando como organizar sua estação de trabalho para trabalhar com [Django](https://www.djangoproject.com/).

**OBSERVAÇÃO IMPORTANTE:**
Todas as informações nesse post foram retiradas através das interações ocorridas no curso [Welcome to the Django](http://welcometothedjango.com.br/),
ministrado pelo [Henrique Bastos](http://henriquebastos.net/).

Primeiramente, é necessário manter algumas coisas atualizadas.
Desta forma, garantimos que alguns comandos não vão falhar.
Observação: algumas dependências não foram instaladas corretamente sem que atualizasse o pip e o setuptools.
Recomendo que isso seja feito **ANTES** e **SEM VIRTUALENV ATIVO**.

```bash
$ pip install -U pip
$ pip install -U setuptools
$ sudo apt-get install python3.5 python3.5-dev git
```

Com as bibliotecas atualizadas, gerar o local onde seu projeto vai estar contido.
Uma observação importante é lembrar que seu projeto não é o projeto do Django. Seu projeto é **maior**!

Portanto, o projeto Django vai estar dentro de uma pasta comum, junto com outros materiais, como, por exemplo,
documentações, scripts, arquivos de configuração, automatizadores de instalação, etc.

Assuma daqui em diante que **project_folder** é um nome decente que identifique seu projeto e que faça sentido!

```bash
$ mkdir project_folder
$ cd project_folder
```

Inicialize um repositório [Git](https://git-scm.com/).
Aconselho inicializá-lo mesmo que o projeto não seja disponibilizado no [Github](https://github.com/),
[Bitbucket](https://bitbucket.org/) ou outro.
Desta forma, você tem um controle de versão e a segurança de poder voltar atrás caso algo de errado.

```bash
$ git init
```

Adicione o arquivo `.gitignore` na pasta contendo o seguinte conteúdo

```
.idea
*.pyc
*.sqlite3
__pycache__
staticfiles
.coverage
.ipynb_checkpoints
```

Crie um [virtualenv](https://virtualenv.readthedocs.org/en/latest/) para isolar o projeto.
Caso esteja utilizando o [virtualenvwrapper](https://virtualenvwrapper.readthedocs.org/en/latest/) e o
[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) (o que recomendo),
mantenha o mesmo nome da pasta e do ambiente virtual, dessa forma, o mesmo será ativado toda vez que entrar na pasta,
eliminando um comando toda vez que for trabalhar.

```bash
$ mkvirtualenv project_folder -p `which python3.5`
$ workon project_folder
```

Para facilitar a interação com o Django, recomendo utilizar o seguinte alias:

```bash
alias manage='python $VIRTUAL_ENV/../manage.py'
```

Hora de instalar alguns pacotes no virtualenv.

Não é necessário fazer tudo isso no início do projeto,
porém algumas dessas bibliotecas são importantes em algum momento do projeto.
Vou listar elas aqui para manter a sanidade quando quiser buscar alguma informação sobre elas.

```bash
$ pip install django
# Duh

$ pip install dj-database-url
# from dj_database_url import parse as dburl

$ pip install python-decouple
# from decouple import config, Csv

$ pip install django-extensions
# manage shell_plus entre outros

$ pip install django-test-without-migrations
# manage test --nomigrations

$ pip install ipython[notebook]
# utilizar pip install "ipython[notebook]" caso use oh-my-zsh
# iPython com módulo notebook

$ pip install prospector
# Ferramenta de qualidade: prospector [options]

$ pip install coverage
# Percentual do código testado: coverage run --source='project_name' manage.py test

$ pip install dj-static
# Cling: Para que seja possível servir arquivos estáticos sem um servidor terceiro como o S3 por exemplo

$ pip install gunicorn
# Server app (Procfile) - Não precisa estar na máquina local

$ pip install psycopg2
# driver do postgres pro python - Não precisa estar na máquina local
```

Por fim, gere o arquivo de requisitos para a virtualenv.
Também aconselho que seja revisado e retirado o que é somente desenvolvimento.

OBSERVAÇÃO: o ideal seria definir o arquivo `requirements.txt` na mão, e depois instalar tudo a partir dele.
Fiz dessa forma pois não sei as versões e não quero deixar no requirements os pacotes sem versão definida.

```bash
pip freeze < requirements.txt
```

Por fim, organizei os arquivos dessa forma:

`requirements.txt`

```
Django==1.9
dj-database-url==0.3.0
dj-static==0.0.6
python-decouple==3.0
static3==0.6.1
gunicorn==19.4.1
psycopg2==2.6.1
coverage==4.0.3
django-extensions==1.6.1
django-test-without-migrations==0.4
```

`requirements-dev.txt`

```
-r requirements.txt
ipython[notebook]
prospector==0.11.7
```

Com os requirements definidos, a instalação pode ser mais simples:

```bash
$ pip install -r requirements-dev.txt
# ou pip install -r requirements.txt
```
