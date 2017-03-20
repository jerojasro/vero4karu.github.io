---
layout: post
title: "Login con Facebook en Django"
date: 2017-03-20 15:24:49 -0500
comments: true
categories: 
- Django
- OAuth
- Facebook
---

Este ejemplo fue probado con la versión de Django 1.10.

Instalamos dos librerías:

```bash
$ pip install social-auth-core
$ pip install social-auth-app-django
```

Agregamos `social_django` en la lista `INSTALLED_APPS` del archivo `settings.py`:

```python
INSTALLED_APPS = [
    'social_django',
]
```

y backend de autenticación:

```python
AUTHENTICATION_BACKENDS = [
    'social_core.backends.facebook.FacebookOAuth2',

    'django.contrib.auth.backends.ModelBackend',
]
```

y al fin los procesadores de contexto:

```python
TEMPLATES = [
    {
        ...
        'OPTIONS': {
            ...
            'context_processors': [
                ...
                'social_django.context_processors.backends',
                'social_django.context_processors.login_redirect',
                ...
            ]
        }
    }
]
```

Ahora podemos correr las migraciónes para crear las tablas que usa `social_django`:

```bash
$ ./manage.py migrate
Running migrations:
  Applying social_django.0001_initial... OK
  Applying social_django.0002_add_related_name... OK
  Applying social_django.0003_alter_email_max_length... OK
  Applying social_django.0004_auto_20160423_0400... OK
  Applying social_django.0005_auto_20160727_2333... OK
  Applying social_django.0006_partial... OK
```

Agregamos urls de `social_django` en `urls.py`:

```python
url('', include('social_django.urls', namespace='social')),
```

Y ahora podemos escribir en la plantilla:

{% codeblock templates/base.html lang:html %}
{% raw %}
<a href="{% url 'social:begin' 'facebook' %}?next={{ request.path }}">Login con Facebook</a>
{% endraw %}
{% endcodeblock %}

Especificamos que queremos asocial usuarios por correo electrónico:

```python
SOCIAL_AUTH_PIPELINE = (
    'social_core.pipeline.social_auth.social_details',
    'social_core.pipeline.social_auth.social_uid',
    'social_core.pipeline.social_auth.auth_allowed',
    'social_core.pipeline.social_auth.social_user',
    'social_core.pipeline.user.get_username',
    'social_core.pipeline.social_auth.associate_by_email',  # <--- este
    'social_core.pipeline.user.create_user',
    'social_core.pipeline.social_auth.associate_user',
    'social_core.pipeline.social_auth.load_extra_data',
    'social_core.pipeline.user.user_details',
)
```

Ir a https://developers.facebook.com y crear una aplicación. Añadir servicio de "Facebook Login":

{% img center /images/django/fb-app-login.png %}

Copiamos las llaves de la aplicación

{% img center /images/django/fb-app-key.png %}

y los pegamos en el archivo de configuración de Django:

{% codeblock settings.py lang:python %}
SOCIAL_AUTH_FACEBOOK_KEY = 'app-id'
SOCIAL_AUTH_FACEBOOK_SECRET = 'app-secret'
{% endcodeblock %}

Después de hacer los pasos anteriores, al momento de hacer click en "Login con Facebook", la aplicación nos redirige a la siguiente página:

{% img center /images/django/fb-login-1.png %}

Pero nosotros, al momento de login queremos obtener correo electrónico de nuestro usuario, por eso definimos scope:

{% codeblock settings.py lang:python %}
SOCIAL_AUTH_FACEBOOK_IGNORE_DEFAULT_SCOPE = True
SOCIAL_AUTH_FACEBOOK_SCOPE = [
    'email'
]
{% endcodeblock %}

Y ahora sí, la aplicación pide acceso al correo electrónico:

{% img center /images/django/fb-login-2.png %}

Entramos al administrador de Django y podemos ver que se creó un registro asociando al usuario que ya existía en la base de tados con su cuenta de FB:

{% img center /images/django/fb-app-django-admin.png %}
