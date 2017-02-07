---
layout: post
title: "Flask con todas las arandelas"
date: 2017-01-20 09:18:28 -0500
comments: true
categories: 
- Flask
- Python
- SQLAlchemy
---

<iframe src="//slides.com/vero4ka/flask-con-todas-las-arandelas/embed?style=light" width="576" height="420" scrolling="no" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

<!-- more -->

## Para empezar

Hay muchos recursos acerca de Flask en internetes, pero aquí me gustaría mencionar los tres que considero fundamentales.

El primero, por supuesto, es la documentación oficial del framework: [flask.pocoo.org/docs](http://flask.pocoo.org/docs/).

Luego, es muy bueno mirar el libro de Miguel Grinberg [flaskbook.com](https://flaskbook.com/), quien de forma bastante detallada esplica cómo hacer una plataforma de publicación de entradas de blog. El cógido, que sirve como ejemplo en el libro, se puede encontrar en la página de github del autor. Como material complemetario al libro, es muy recomendado echar una meirada en su blog personal de [blog.miguelgrinberg.com](https://blog.miguelgrinberg.com/), donde Miguel Grinberg cubre los temas más específicas de desarrollo en Flask.

Y ahora, como dijo el austronauta ruso Yuri Gagarin en el momento del despegue de su nave Vostok 1: "¡Poyejali!"" (en ruso: Поехали!; se traduce como «¡Vámonos!»).

## Aplicación básica

Empezamos con instalar **Flask**:

```
$ mkvirtualenv pycon2017 -p python3
$ pip install Flask
```

Al moento de ésta presentación la últma versión del framework es **0.12**.

Ahora creamos nuesta primera aplicancíon de Flask que va a ser sólo un archivo de Python:

{% codeblock app.py lang:python %}
from flask import Flask
app = Flask(__name__)


@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
{% endcodeblock %}

El decorador ``route`` nos auyda a decinir la ruta de URL para la vista llamada ``hello`` que simplemtnte devuelve al navegador la respuesta HTTP 200 con la frase "Hello World!". 

Corrimos la aplicación con el siguiente comando:

```
$ python app.py 
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

Y ahora lo podemos abrir en nuestro navegador:

{% img center /images/flask/flask_01.png %}

## manage.py

Nosotros, por experiencia con otros frameworks, ya nos acostumbramos con la posibilidad de acceder shell de Python desde nuestra aplicación, o con poder correr comandos que pertenecen a la aplicación desde la consola. Para tener todo esto en Flask, instalamos un librería llamada `Flask-Script`.

```
$ pip install Flask-Script
```

Separamos nuesta aplicación en dos archovos: en el primero vamos iniciar la app de Flask, en la que especificamos la ruta de los archivos estáticos y el archovo de configuración (equivalente de `settings.py` en Django)

{% codeblock run.py lang:python %}
from flask import Flask

def create_app():
    app = Flask(__name__, 
        static_url_path='/static')
    app.config.from_object('conf.config')
    return app
{% endcodeblock %}

y en el segundo vamos a colocar la instancia de **Manager**

{% codeblock manage.py lang:python %}
from flask.ext.script import Manager
from run import create_app

app = create_app()
manager = Manager(app)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == '__main__':
    manager.run()
{% endcodeblock %}

Corriendo el sigiente comando en la consola

```
$ ./manage.py runserver -h HOST -p PORT
```

podemos ver la misma aplicación, como en el paso anterior, sino ahora podemos acceder a shell usando el comando

```
$ ./manage.py shell
In [1]:
```

## Configuración

Miramos con más atención la estructura del proyecto:

```
├── conf
│   ├── config.py
│   ├── __init__.py
│   └── local_config.py
├── manage.py
└── run.py
```

Ahora tenemos una carpeta con los archivos de configuración - `conf`:

{% codeblock conf/config.py lang:python %}
import sys

DEBUG = False

try:
    if 'test' in sys.argv:
        from test_config import *
    else:
        from local_config import *
except ImportError:
    pass
{% endcodeblock %}

Más información acerca de la configuración en Flask se puede encontrar en la página [flask.pocoo.org/docs/0.12/config/](http://flask.pocoo.org/docs/0.12/config/).

## Blueprints

Cuando nuesto archivo con las vistas crezca, vamos a querer a separar la lógica de nuestra aplicación en los modulos diferentes (así como lo tenemos con apps de Django):

```
├── conf
│   ├── config.py
│   ├── __init__.py
│   └── local_config.py
├── app1
│   ├── templates
│   └── views.py
├── app2
│   ├── templates
│   └── views.py
├── manage.py
└── run.py
```

En este caso nos auyda un consepto de crear aplicaciones llamado **Blueprints**. La informacion completa acerca de Blueprints se puede encontrar en la documentación de Flask: [flask.pocoo.org/docs/0.12/blueprints/](http://flask.pocoo.org/docs/0.12/blueprints/). 

Creamos un Blueprina para nuesta aplicación, conde vamos a guardad las vistas relacionadas a nuestros usuarios, por ejemplo:

{% codeblock users/views.py lang:python %}
from flask import Blueprint

users = Blueprint(
    'users',
    __name__,
    template_folder='templates')

@users.route('/')
def hello():
    return 'Hello Worlds!'
{% endcodeblock %}

y lo registaramos en el archivo `run.py`:

{% codeblock run.py lang:python %}
from flask import Flask


def create_app():
    app = Flask(__name__,
        static_url_path='/static')
    app.config.from_object('conf.config')

    from users.views import users
    app.register_blueprint(users, url_prefix='/users')

    return app
{% endcodeblock %}

indicando en el argumento `url_prefix` del método `register_blueprint`, que todos los URLs, relacionadas con el Blueprint de usuario, van a empezar con el prefijo `/users`, por ejemplo: [http://localhost:5000/users/](http://localhost:5000/users/).

## URLs

Miramos cómo se definen los URLs en Flask.

Primero, hay formas diferentes de espesificar la ruta: usando el decorador de Blueprint `@users.route` o el método `users.add_url_rule`:

```python
@users.route('/user/<name>')
def user_profile(name):
    return '<h1>Hello, {}!</h1>'.format(name)

def users_list():
    pass
users.add_url_rule('/', 'users_list', users_list)
```

En el primer ejemplo podemos ver, que a la vista se para un paramentro de URL `name`, que luego accedemos desde el parametro `name` de la vista `user_profile`.

Flaks permite especificar el típo de parametro que esperamos. Por ejemplo, para los númetros enteros, el decorador de escribe así:

```python
@users.route('/user/<int:pk>')
def user_detail(pk):
    return '<h1>Hello, user #{}!</h1>'.format(pk)
```

También se puede eplicitamente explicar el método HTTP, a que responde la vista.

```python
@users.route('/user/<int:pk>/create', methods=('POST',))
def user_create(pk):
    pass
```

En este caso al tratar de acceder la vista con la petición GET, obtenemos el error 405 (Method not allowed).

Para obtener URL, que corresponde a la vista usamos el método `url_for`, al que pasamos el nombre de Blueprint y nombre de la vista:

```python
from flask import url_for

url_for('users.user_detail', pk=user_pk)
```

## Vistas