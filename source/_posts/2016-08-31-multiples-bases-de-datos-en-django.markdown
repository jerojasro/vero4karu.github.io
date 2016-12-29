---
layout: post
title: "Múltiples bases de datos en Django"
date: 2016-08-31 11:15:01 -0500
comments: true
categories: 
- Django
---

Tenemos dos tablas con esquemas iguales dos en bases de datos diferentes: `production` y `history`. 

```python settings.py
DATABASE_ROUTERS = ['routers.HistoricRouter']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': 'localhost',
        'NAME': 'production',
        'USER': 'secret',
        'PASSWORD': 'secret',
    },
    'historical': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': 'localhost',
        'NAME': 'history',
        'USER': 'secret',
        'PASSWORD': 'secret',
    }
}
```

Escribimos un router para definir a qué base de datos hacer la petición:

```python routers.py
class HistoricRouter(object):
    """
    A router to control all database operations on models in the historic application.
    """
    def db_for_read(self, model, **hints):
        """
        Attempts to read historic models go to historical.
        """
        if model._meta.app_label == 'historic':
            return 'historical'
        return None

    def db_for_write(self, model, **hints):
        """
        Attempts to write historic models go to historical.
        """
        if model._meta.app_label == 'historic':
            return 'historical'
        return None

    def allow_relation(self, obj1, obj2, **hints):
        """
        Allow relations if a model in the historic app is involved.
        """
        if obj1._meta.app_label == 'historic' or \
            obj2._meta.app_label == 'historic':
            return True
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        """
        Make sure the historic app only appears in the 'historical' database.
        """
        if app_label == 'historic':
            return db == 'historical'
        return None
```

En nuestros modelos especificamos `app_label` para referirnos a la base de datos distinta a `default`:

```python models.py
class Author(models.Model):

    class Meta:
        db_table = 'authors'

class AbstractBook(models.Model):
    author = models.ForeignKey('books.Author', related_name='books')
    created_at = models.DateTimeField()

    class Meta:
        unmanaged = True
        abstract = True

class Book(Book):

    class Meta:
        db_table = 'books'

class HistoricBook(Book):

    class Meta:
        unmanaged = True
        db_table = 'books'
        app_label = 'historic'
```

La petición `Book.objects.all()` va a traer libros de la base de datos `production` y `HistoricBook.objects.all()` - de la base de datos `history`.

Modificando `allow_relation` en nuestro router, podemos permitir llaves foráneas entre modelos de diferentes bases de datos:

```python
    def allow_relation(self, obj1, obj2, **hints):
        return True
```

Enlaces:

* [Django: Cross-database relations](https://docs.djangoproject.com/en/1.10/topics/db/multi-db/#cross-database-relations)