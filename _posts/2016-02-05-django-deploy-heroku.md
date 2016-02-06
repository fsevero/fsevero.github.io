---
layout: post
title: Deploy do projeto Django
---
"Tutorial" explicando como enviar o nosso projeto [Django](https://www.djangoproject.com/) 
para produção, utilizando o [Heroku](https://www.heroku.com/).

**OBSERVAÇÃO IMPORTANTE:**
Todas as informações nesse post foram retiradas através das interações ocorridas no curso [Welcome to the Django](http://welcometothedjango.com.br/),
ministrado pelo [Henrique Bastos](http://henriquebastos.net/).

Organizamos a estrutura para o projeto, criamos o projeto Django, criamos uma landing page. E agora? Precisamos colocar no ar!

Neste post, vou repassar brevemente o processo para deploy no Heroku. Outras plataformas serão adicionadas com o tempo.

Antes de fazer o primeiro deploy, precisamos de algumas estruturas auxiliares.

Crie o arquivo `Procfile`, com o seguinte conteúdo:

```
web: gunicorn project_name.wsgi --log-file -
```

Cuidado com a escrita, o nome do arquivo deve ser **EXATAMENTE IGUAL**, primeira letra em maiúscula e sem extensão.
Muitos erros no momento do deploy são causados por este arquivo não estar com o nome ou o conteúdo correto!
Lendo esse arquivo o Heroku detecta que a gente criou um **processo web** e qual o **comando para inicializá-lo**.

Outro arquivo que precisa ser criado é o `runtime.txt`, o qual definirá qual a versão do python deve ser executada
(Por padrão é a 2.7). O conteúdo do arquivo deve ser:

```
python-3.5.0
```

Com estes arquivos configurados, vamos instalar o [Heroku toolbet](https://toolbelt.heroku.com/).
Este pacote possui alguns facilitadores para realizar o deploy no sistema do heroku.
[Neste site](https://toolbelt.heroku.com/), existe uma explicação de como instalá-lo em cada um dos principais sistemas operacionais.
Para um sistema Ubuntu ou compatível, basta rodar o comando:

```bash
$ wget -qO- https://toolbelt.heroku.com/install-ubuntu.sh | sh
```

Após completar, teremos disponível o comando heroku.
Rode o seguinte comando para verificar a instalação e atualizar o toolbelt
(Sim, acabamos de instalá-lo, mas podemos precisar de atualização).

```bash
$ heroku
```

Após, tudo instalado e atualizado, precisamos “logar” na nossa conta do heroku (a qual deve ter sido previamente criada pelo site):

```bash
$ heroku login
# siga as orientações
```

Agora, devemos criar uma nova aplicação no Heroku:

```bash
$ heroku apps:create app_name
# Estará disponível como http://app_name.herokuapp.com
 
# Adiciona nossas variáveis de ambiente (.env) para o heroku
$ heroku config:push
# talves seja necessário rodar:
# heroku plugins:install git://github.com/ddollar/heroku-config.git
 
# Após ter mandado os valores padrões, variáveis podem ser adicionadas
# ou alteradas com o comando
# heroku config:set key=value
```

Agora, basta enviar seu código para sua app, com o comando:

```bash
$ git push heroku master --force
```

Outros comandos podem ser executados na sua instância do Heroku com:

```bash
$ heroku run command
# heroky run python manage.py migrate
```

Para facilitar o deploy da aplicação, criei a seguinte função, a qual deve ser adicionada ao `.bash_aliases`,
`.profile`, ou em algum lugar onde você define seus alias e funções:

```bash
heroku_deploy () {
  # Habilita modo manutenção, o qual impedirá o acesso ao site
  # através de uma mensagem de "em manutenção"
  heroku maintenance:on;
  git push heroku master;
  heroku run python manage.py migrate;
  heroku maintenance:off;
}
```
