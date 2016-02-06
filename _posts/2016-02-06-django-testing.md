---
layout: post
title: Iniciando um projeto com o Django
---
Este post irá conter uma "coletânea" de formas de testar seu código [Django](https://www.djangoproject.com/).
Ele não será um passo a passo de como fazer TDD ou testes de regressão.
Mas um conjunto de "blocos de código" para servir de inspiração na hora de desenhar seus testes.

**OBSERVAÇÃO IMPORTANTE:**
Todas as informações nesse post foram retiradas através das interações ocorridas no curso [Welcome to the Django](http://welcometothedjango.com.br/),
ministrado pelo [Henrique Bastos](http://henriquebastos.net/).

#### Um teste básico

Basicamente um caso de teste é composto por um método de "set up", alguns testes, e um método de "rollback".
O setUp e o tearDown não são obrigatórios e só devem ser utilizados se forem realmente necessários.

```python
from django.test import TestCase

class UseCaseTest(TestCase):
    def setUp(self):
        # organize variáveis e chamadas padrões aqui
        # self.response = ...
        # self.obj = ...
        
    def test_some_func(self):
        # Primeiro teste do caso de testes
        
    def test_some_other_stuff(self):
        # Segundo teste
        
    def tearDown(self):
        # Lógica posterior à execução dos testes
        # rollback...
```

#### Testando uma view (GET)

Exemplo de testes para um GET em uma view:

```python
from django.test import TestCase
from django.shortcuts import resolve_url as r

from path.to.models import User
from path.to.forms import SubscriptionForm

class ViewUseCaseTest(TestCase):
    def setUp(self):
        self.obj = User.objects.create(dict())
        self.response = self.client.get(r('home'))
        self.response_with_object = self.client.get(r('detail', obj))
        # Normalmente esses dois "gets" estariam em dois casos de teste diferentes
        # Estão juntos aqui somente para exemplificação
        
    def test_status_code(self):
        self.assertEqual(200, self.response.status_code)
        
    def test_template_used(self):
        self.assertTemplateUsed(self.response, 'path/to/template.html')
        
    def test_context(self):
        user = self.response.context['user']
        self.assertTrue(user)
        self.assertIsInstance(user, User)
        
    def test_html_content(self):
        contents = ['Severo', '<a href=', 'Título da página']
        
        for content in contents:
            with self.subTest():
                self.assertContains(self.response, content)
                
        contents = [
            (1, 'Título da página'),
            (5, '<a href='),
        ]
        
        for count, content in contents:
            with self.subTest():
                self.assertContains(self.response, content, count)
                
    def test_not_found(self):
        response = self.client.get(r('detail', 0))
        self.assertEqual(404, response.status_code)
        
    def test_csrf(self):
        """ HTML must contain csrf"""
        self.assertContains(self.response, 'csrfmiddlewaretoken')

    def test_has_form(self):
        """ Context must have subscription form"""
        form = self.response.context['form']
        self.assertIsInstance(form, SubscriptionForm)
        
    def test_subscription_link(self):
        expected = 'href="{}"'.format(r('subscriptions:new'))
        self.assertContains(self.response, expected)
```

#### Testando uma view (POST)

Exemplos de teste para um POST em uma view:

```python
from django.test import TestCase
from django.shortcuts import resolve_url as r
from django.core import mail

from path.to.models import Subscription

class PostValidUseCaseTest(TestCase):
    def setUp(self):
        data = dict()
        self.response = self.client.post(r('subscriptions:new'), data)

    def test_post(self):
        """Valid POST should redirect to /subscriptions/1/ """
        self.assertRedirects(self.response, r('subscriptions:detail', 1))

    def test_subscribe_email(self):
        """Valid POST should send ONE email"""
        self.assertEqual(1, len(mail.outbox))

    def test_save_subscription(self):
        """Valid POST should create an object"""
        self.assertTrue(Subscription.objects.exists())


class PostInvalidUseCaseTest(TestCase):
    def setUp(self):
        self.response = self.client.post(r('subscriptions:new'), {})

    def test_post(self):
        """Invalid POST should not redirect"""
        self.assertEqual(200, self.response.status_code)

    def test_template(self):
        """Invalid POST should have same template"""
        self.assertTemplateUsed(self.response, 'subscriptions/subscription_form.html')

    def test_has_form(self):
        """Invalid POST must have a form"""
        form = self.response.context['form']
        self.assertIsInstance(form, SubscriptionForm)

    def test_has_errors(self):
        """Invalid POST must have a form with errors"""
        form = self.response.context['form']
        self.assertTrue(form.errors)

    def test_dont_save_subscription(self):
        """Invalid POST can't create and object"""
        self.assertFalse(Subscription.objects.exists())
        
    def test_template_has_non_field_errors(self):
        invalid_data = dict()
        response = self.client.post(r('subscriptions:new'), invalid_data)
        self.assertContains(response, '<ul class="errorlist nonfield">')
```

#### Testando envio de email

Exemplos de teste para envio de email:

```python
from django.core import mail
from django.test import TestCase
from django.shortcuts import resolve_url as r


class EmailUseCaseTest(TestCase):
    def setUp(self):
        data = dict()
        self.client.post(r('subscriptions:new'), data)
        self.email = mail.outbox[0]

    def test_subscription_email_subject(self):
        expect = 'Confirmação de inscrição'
        self.assertEqual(expect, self.email.subject)

    def test_subscription_email_from(self):
        expect = 'email@example.com.br'
        self.assertEqual(expect, self.email.from_email)

    def test_subscription_email_to(self):
        expect = ['email@example.com.br', 'example@email.com']
        self.assertEqual(expect, self.email.to)

    def test_subscription_email_body(self):
        contents = []
        for content in contents:
            with self.subTest():
                self.assertIn(content, self.email.body)
```

#### Testando models

Exemplos de teste para models, managers, etc:

```python
from datetime import datetime
from django.shortcuts import resolve_url as r
from django.test import TestCase
from path.to.models import Subscription, Contact, Speaker
from path.to.managers import ContactManager


class SubscriptionModelTest(TestCase):
    def setUp(self):
        self.obj = Subscription(
            name="Fabrício Severo"
        )
        self.obj.save()

    def test_create(self):
        self.assertTrue(Subscription.objects.exists())

    def test_created_at(self):
        """Subscription mus have an auto created_at attr."""
        self.assertIsInstance(self.obj.created_at, datetime)

    def test_str(self):
        self.assertEqual('Fabrício Severo', str(self.obj))

    def test_some_default_value(self):
        """By default, paid must be false"""
        self.assertEqual(False, self.obj.paid)

    def test_absolute_url(self):
        """get_absolute_url must be implemented"""
        url = r('subscriptions:detail', self.obj.pk)
        self.assertEqual(url, self.obj.get_absolute_url())
        
    def test_relation_with_other_model(self):
        contact = Contact.objects.create(subscription=self.obj, value='email@example.com')
        self.assertTrue(Contact.objects.exists())
        
    def test_validation(self):
        """Contact kind should be limited to E or P"""
        contact = Contact(kind='A', value='B')
        self.assertRaises(ValidationError, contact.full_clean)
        
    def test_description_can_be_blank(self):
        field = Subscription._meta.get_field('description')
        self.assertTrue(field.blank)
        
    def test_description_can_be_null(self):
        field = Subscription._meta.get_field('description')
        self.assertTrue(field.null)
        
    def test_has_contacts(self):
        """Subscription has contacts"""
        self.obj.contacts.create(dict())
        self.assertEqual(1, self.obj.contacts.count())
        
    def test_ordering(self):
        self.assertListEqual(['created'], Subscription._meta.ordering)
        self.assertListEqual(['-created'], Subscription._meta.ordering)
        
class ContactManagerTest(TestCase):
    def setUp(self):
        s = Speaker.objects.create()
        s.contact_set.create(kind=Contact.EMAIL, value='email@example.com')
        s.contact_set.create(kind=Contact.PHONE, value='00-111112222')
        
    def test_manager(self):
        self.assertIsInstance(Contact.objects, ContactManager)

    def test_emails(self):
        qs = Contact.objects.emails()
        expected = ['email@example.com']
        self.assertQuerysetEqual(qs, expected, lambda o: o.value)

    def test_phones(self):
        qs = Contact.objects.phones()
        expected = [00-111112222]
        self.assertQuerysetEqual(qs, expected, lambda o: o.value)
```
