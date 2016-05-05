---
layout: post
title: "Pruebas para formularios en Django"
date: 2015-09-30 11:49:19 +0200
comments: true
categories: Django TDD
---

En esta entrada de blog quiero compartir la forma en la que escribo ruebas para formularios en un proyecto de Django.

Primero vamos a instalar los paquetes necesarios:

{% codeblock %}
$ pip install django-webtest
$ pip install factory-boy
{% endcodeblock %}

[WebTest](http://webtest.pythonpaste.org/en/latest/WebTest) es una biblioteca que nos ayuda a escribir pruebas para las aplicaciones wsgi. Es mucho más poderosa comparando con ``django.test.Client`` que viene con Django.

En el archivo de configuración especificamos que vamos a usar la base de datos SQLite para correr nuestras pruebas:

{% codeblock test_settings.py lang:python %}
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
    }
}
{% endcodeblock %}

Suponemos que queremos escribir una prueba unitaria para una vista que crea un objeto del modelo ``Event``:

{% codeblock models.py lang:python %}
class Event(models.Model):
    title = models.CharField(max_length=500)
    start_date = models.DateTimeField()
    end_date = models.DateTimeField(blank=True, null=True)
    people = models.ManyToManyField('history.Person', blank=True, related_name='events')

class Person(models.Model):
    name = models.CharField(max_length=255)
{% endcodeblock %}

Aquí está la vista que maneja el formulario para crear un evento:

{% codeblock views.py lang:python %}

class EventCreateView(LoginRequiredMixin, CreateView):
    model = Event
    form_class = EventForm
    success_url = reverse_lazy('history:timeline')
{% endcodeblock %}

Ahora escribimos nuestra prueba. Vamos a extender la clase ``django_webtest.WebTest``, que en su lugar extiende ``django.test.TestCase`` de Django, y crear a un usuario:

{% codeblock tests.py lang:python %}
from django_webtest import WebTest

class HistoryViewsTests(WebTest):
    def setUp(self):
        self.superuser = get_user_model().objects.create_superuser(
            email='superuser@example.com',
            username='superuser',
            password='secret',
        )
        super(HistoryViewsTests, self).setUp()

    def test_create_event(self):
        # código para nuestra prueba
{% endcodeblock %}

Ahora podemos probar que cuando un usuario trata de acceder nuestra página, se redirige al formulario de acceso (el códigos de estado HTTP es 302):

{% codeblock lang:python %}
url = reverse('history:event_create')
self.app.get(url, status=302)
{% endcodeblock %}

Ahora vamos a acceder la vista como superusuario:

{% codeblock lang:python %}
response = self.app.get(url, user=self.superuser, status=200)
{% endcodeblock %}

Como tenemos solo un formulario en la página, podemos obtenerlo por una de tres formas: por el atributo ``.form``: ``response.form``, por índice ``response.forms[0]`` o por ``id`` del formulario (artibuto HTML)`` response.forms['event_form']``.

Para facilitar la depuración de nuestras pruebas, podemos pedir a mostrar el ``response`` en navegador predeterminado:

{% codeblock lang:python %}
response.showbrowser()
{% endcodeblock %}

Ahora podemos diligenciar nuestro formulario con los datos de prueba:

{% codeblock tests.py lang:python %}
class HistoryViewsTests(WebTest):
    # ...

    def test_create_event(self):
        person1, person2, person3 = PersonFactory.create_batch(3)
        response = self.app.get(url, user=self.superuser, status=200)

        self.assertFalse(Event.objects.exists())

        form = response.forms['event_form']
        form['title'] = u'Título del evento'
        form['start_date'] = datetime.datetime.now()
        form['people'] = [person1.id, person2.id]
        form.submit().follow()

        self.assertEqual(Event.objects.count(), 1)

        event = Event.objects.first()
        self.assertEqual(event.title, u'Título del evento')
        
        people = event.people.all()
        self.assertIn(person1, people)
        self.assertIn(person2, people)
        self.assertNotIn(person3, people)
{% endcodeblock %}

Para generar objetos del modelo ``Person`` usamos la biblioteca [Factory Boy](http://factoryboy.readthedocs.org/):

{% codeblock factories.py lang:python %}
from ..models import Person

import factory
import factory.fuzzy

class PersonFactory(factory.django.DjangoModelFactory):
    name = factory.fuzzy.FuzzyText(length=50)

    class Meta:
        model = Person
{% endcodeblock %}

Así podemos acceder los opciones de una selección o selección múltiple:

{% codeblock lang:pycon %}
>>> print form['people'].options
[(u'1', False, u'GLUZLvZfyjdZEmjNtnAJvsIVljodQjZpzLRDKrqJtYGiDLmSrN'), (u'2', False, u'vOxDBbmLaUXxJkJzcqYgLQpBieSoLtXJcpHCEPUpYUIzybhsAh'), (u'3', False, u'tfXSXCTIQICDwVPYvxZGSXgclFTnHbeYSQaMntxJNcgUJjzAwX')]
>>> form['people'] = [person1.id, person2.id]
>>> print form['people'].value
[u'1', u'2']
{% endcodeblock %}

Ahora corremos nuestras pruebas y voilà:

{% codeblock %}
$ ./manage.py test
Creating test database for alias 'default'...
..
----------------------------------------------------------------------
Ran 1 test in 0.463s

OK
Destroying test database for alias 'default'...
{% endcodeblock %}

Enlaces:

* [WebTest](https://github.com/django-webtest/django-webtest)
* [Manejo de formularios con WebTest](http://webtest.pythonpaste.org/en/latest/forms.html)
* [Factory Boy](http://factoryboy.readthedocs.org/)