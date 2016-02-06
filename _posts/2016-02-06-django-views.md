---
layout: post
title: Django - TEST FIRST
---
Este post irá conter uma "coletânea" de formas de criar views no [Django](https://www.djangoproject.com/).
Ele não será um passo a passo de como fazer views.
Mas um conjunto de "blocos de código" para servir de inspiração na hora de desenhar suas views.

**OBSERVAÇÃO IMPORTANTE:**
Todas as informações nesse post foram retiradas através das interações ocorridas no curso [Welcome to the Django](http://welcometothedjango.com.br/),
ministrado pelo [Henrique Bastos](http://henriquebastos.net/).

#### Hello World!
```python
from django.http import HttpResponse
def hello(request):
  return HttpResponse("Hello World")
```

#### View simples para renderizar um template
```python
from django.shortcuts import render

def home(request):
    return render(request, 'index.html')

def other(request):
    objs = Model.objects.all()
    return render(request, 'list.html', {'data': objs})
```

#### View que envia um email
```python
from django.core import mail
from django.shortcuts import render
from django.template.loader import render_to_string


def view_that_sends_emails(request):
  body = render_to_string('path/to/email_model.txt', dict())
  mail.send_mail('Título', body, 'from@email.com', ['to@emails.com'])
  return render(request, 'index.html')
```

#### View que renderiza uma mensagem no sessão
```python
from django.contrib import messages
from django.shortcuts import render

def view_with_message(request):
  messages.success(request, 'Essa mensagem estará disponível em {{ messages }} !')
  return render(request, 'index.html')
```

#### View que redireciona
```python
from django.shortcuts import render
from django.http import HttpResponseRedirect
from django.shortcuts import resolve_url as r

def view_with_message(request):
  some_validation = True
  if some_validation:
    return HttpResponseRedirect(r('somewhere'))
    
  return render(request, 'index.html')
```

#### View "roteadora"
```python
def subscribe(request):
    if request.method == 'POST':
        return create(request)
    else:
        return new(request)
```

#### View que apresenta um objeto do banco de dados
```python
from django.shortcuts import render, get_object_or_404
from eventex.core.models import Speaker, Subscription
from django.http import Http404

def detail(request, pk):
    try:
        subscription = Subscription.objects.get(pk=pk)
    except Subscription.DoesNotExist:
        raise Http404

    return render(request, 'subscriptions/subscription_detail.html',
                  {'subscription': subscription})
                  
def speaker_detail(request, slug):
    speaker = get_object_or_404(Speaker, slug=slug)
    return render(request, 'core/speaker_detail.html',
                  {'speaker': speaker})
```

#### Generic Views
```python
from django.views.generic import ListView, DetailView
from eventex.core.models import Speaker, Talk

home = ListView.as_view(template_name='index.html', model=Speaker)
# core/index.html

speaker_detail = DetailView.as_view(model=Speaker)
# core/speaker_detail.html
# {{ app_name}}/{{ model_name }}_detail.html

talk_list = ListView.as_view(model=Talk)
# app_name/talk_list.html
# {{ app_name}}/{{ model_name }}_list.html
```
