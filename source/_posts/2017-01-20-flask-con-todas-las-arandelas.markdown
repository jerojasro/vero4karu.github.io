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

Hay muchos recursos acerca de Flask en los internetes, pero aquí me gustaría mencionar los tres que considero fundamentales.

El primero, por supuesto, es la documentación oficial del framework: [flask.pocoo.org/docs](http://flask.pocoo.org/docs/).

Luego, es muy bueno mirar el libro de Miguel Grinberg [flaskbook.com](https://flaskbook.com/), quien de forma bastante detallada esplica cómo hacer una plataforma de publicación de entradas de blog. El cógido, que sirve como ejemplo en el libro, se puede encontrar en la página de github del autor. Como material complemetario al libro, es muy recomendado echar una mirada en su blog personal de [blog.miguelgrinberg.com](https://blog.miguelgrinberg.com/), donde Miguel Grinberg cubre los temas más específicas de desarrollo en Flask.

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

#### Request

En Flask el objeto `request` no se está pasando como parametro al la vista (como en Django, por ejemplo), sino es una variable global:

```
GET http://localhost:5000/users/user/1?q=foo
```

{% codeblock users/views.py lang:python %}
from flask import Blueprint, request

users = Blueprint('users', __name__, template_folder='templates')


@users.route('/user/<int:pk>')
def user_detail(pk):
    print(request)
    print(request.method)  # GET
    print(request.args)    # ImmutableMultiDict([('q', u'foo')])
    print(request.headers)
    return '<h1>Hello, user #{}!</h1>'.format(pk)
{% endcodeblock %}

#### Decoradores

Aparte de la definición de ruta, la vista en Flask puede tenet otros decoradores, por

{% codeblock users/views.py lang:python %}
@users.route('/user/<int:pk>')
@login_required
def user_detail(pk):
    return '<h1>Hello, user #{}!</h1>'.format(pk)
{% endcodeblock %}

## Plantillas

Ahora vamos a renderizar una plantilla desde nuestra vista. Para eso vamos a usar el método `render_template` que recibe como argumentos la ruta hasta la plantilla dentro de la carpeta que especificamos en el parametro `template_folder` del Blueprint, y los demás argumentos que son los parametros de contexto.

{% codeblock users/views.py lang:python %}
from flask import Blueprint
from flask import render_template

users = Blueprint('users', __name__, template_folder='templates')


@users.route('/user/<int:pk>')
def user_detail(pk):
    return render_template('users/detail.html', user_id=pk)
{% endcodeblock %}

{% codeblock users/templates/users/detail.html lang:html %}
{% raw %}
<html>
  <body>
    <h1>Hello, user #{{ user_id }}!</h1>
  </body>
</html>
{% endraw %}
{% endcodeblock %}

### Jinja2

Para renderizar plantillas Flask usa el lenguaje `Jinja2`. Vamos a mirar algunas de sus funcionalidases:

#### Variables:

{% codeblock lang:html %}
{% raw %}
{{ foo.bar }}
{{ foo['bar'] }}
{% endraw %}
{% endcodeblock %}

#### Condicionales:

{% codeblock lang:html %}
{% raw %}
{% if user.address %}
    <p>{{ user.address }}</p>
{% endif %}
{% endraw %}
{% endcodeblock %}

##### Bucles:

{% codeblock lang:html %}
{% raw %}
{% for user in users %}
  <li>{{ user.name }}</li>
{% else %}
  <li>No hay usuarios</li>
{% endfor %}
{% endraw %}
{% endcodeblock %}

##### Bloques:

{% codeblock lang:html %}
{% raw %}
{% block title %}Usuarios{% endblock %}
{% endraw %}
{% endcodeblock %}

##### Extender una plantilla base:

{% codeblock lang:html %}
{% raw %}
{% extends "layouts/main.html" %}
{% endraw %}
{% endcodeblock %}

#### Comentario:

{% codeblock lang:html %}
{% raw %}
{# Texto #}
{% endraw %}
{% endcodeblock %}

##### Incluir otra plantilla:

{% codeblock lang:html %}
{% raw %}
{% with rating=user.rating %}
    {% include 'includes/_rating.html' %}
{% endwith %}
{% endraw %}
{% endcodeblock %}

##### Asignar valor a una variable local:

{% codeblock lang:html %}
{% raw %}
{% set permissions = user.get_permission() %}
{{ permissions }}
{% endraw %}
{% endcodeblock %}

##### Filtros:

{% codeblock lang:html %}
{% raw %}
{{ user.address|default('N/A') }}
{% endraw %}
{% endcodeblock %}

## Procesadores de contexto

De forma predetermiada, el contexto de todas las plantillas ya tienen los sigientes variables:

* `config` - Objeto de configuración (`flask.config`)

{% codeblock lang:html %}{% raw %}
{{ config.DEBUG }}
{% endraw %}{% endcodeblock %}

* `request` - Objeto de la petición actual (`flask.request`)

{% codeblock lang:html %}{% raw %}
{{ request.path }}
{% endraw %}{% endcodeblock %}

* `session` - Objeto de sesión (`flask.session`)
* `g` - Variables globales

Hay una forma de tener `context_processors` (como en Django) par apoder pasar unas variables a todas las plantillas del proyecto:

{% codeblock run.py lang:python %}
from flask import Flask

def create_app():
    app = Flask(__name__, static_url_path='/static')
    app.config.from_object('conf.config')

    from users.views import users
    app.register_blueprint(users, url_prefix='/users')

    @app.context_processor
    def constants_processor():
        return {
            'say_hello': 'Hola',
        }

    return app
{% endcodeblock %}

Ahora podemos acceder la variable `say_hello` desde todas las plantillas sin tener que pasarla cada vez explicitamente:

{% codeblock users/templates/users/detail.html lang:html %}
{% raw %}
<html>
  <body>
    <h1>{{ say_hello }}, user #{{ user_id }}!</h1>
  </body>
</html>
{% endraw %}
{% endcodeblock %}

## Modelos

### SQLAlchemy

Hay dos librerías que nos permiten trabajar con modelos y hacer periciones SQL desde Flask.

```
$ pip install SQLAlchemy
$ pip install Flask-SQLAlchemy
```

Una es `SQLAlchemy` y la otra es su extención para Flask - `Flask-SQLAlchemy`, que viene con algunas funcionalidades adicionales y útiles en el desarrollo web, por ejemplo, el método `first_or_404()` para obtener el primer elemento del query o error HTTP 404 si no existe, o el método `paginate()` para realizar paginación sobre los objetos de `BaseQuery`.

Ahora incluimos `SQLAlchemy` como una aplicación externa de nuestro proyecto. Primero creamos una varible `db` para que el import no se rompa cuando la aplicación todavía no se inicializó, y en `create_app` y inicializamos la app: `db.init_app(app)`.

{% codeblock run.py lang:python %}
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__, static_url_path='/static')
    app.config.from_object('conf.config')

    db.init_app(app)
    return app
{% endcodeblock %}

En el archivo de comfiguración colocamos la ruta hacía nuestra base de datos.

{% codeblock conf/config.py lang:python %}
SQLALCHEMY_DATABASE_URI = 'sqlite:////tmp/test.db'
{% endcodeblock %}

Y ahora posemos crear los modelos. Aquí en ejemplo se puede ver comómo definir modelos, columnas de tipos diferentes, crear llaves foráneas

{% codeblock users/models.py lang:python %}
from run import db


class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)
    city_id = db.Column(db.ForeignKey(u'cities.id'), nullable=False, index=True)
    created_at = db.Column(db.DateTime, nullable=False, default=db.func.now())
    updated_at = db.Column(db.DateTime, nullable=False, default=db.func.now(),
                        onupdate=db.func.now())

    def __repr__(self):
        return '<Usuario %r>' % self.username


class City(db.Model):
    __tablename__ = 'cities'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(255))

    users = db.relationship('User', backref='city', lazy='dynamic')
{% endcodeblock %}

### CRUD

Miramos los métodos de CRUD básicos que nos ofrece el ORM de SQLAlchemy.

##### Crear

{% codeblock lang:python %}
user = User(email='john@example.com', username='')
db.session.add(user)
db.session.commit()
{% endcodeblock %}

##### Obtener todos usuarios

{% codeblock lang:python %}
users = User.query.all()  # [<Usuario john'>, <Usuario u'admin'>]
{% endcodeblock %}

##### Obtener el primer usuario

{% codeblock lang:python %}
john = User.query.first()  # <Usuario 'john'>
{% endcodeblock %}

##### Filtrar usuarios y ordenar

{% codeblock lang:python %}
User.query.filter_by(username='john').all()
User.query.filter(
    User.created_at >= (datetime.datetime.utcnow() - datetime.timedelta(days=3)
).order_by(User.created_at.desc()).limit(10).all()
{% endcodeblock %}

##### Borrar

{% codeblock lang:python %}
db.session.delete(user)
db.session.commit()
{% endcodeblock %}

`BaseQuery` no tiene metodo `save()`, sino todos los cambios que hacemos a objetos de modelos se agraga a las sessión. Para realizar la transacción correspondiente a la base de datos, hace falta llamar un método `db.session.commit()`.

## Formularios

Existen varisas librerías de Python que nos permiten trabajar con formularios. En éste ejemplo vamos a mirar una llamada `Flask-WTF`.

```
$ pip install Flask-WTF
$ pip install Flask-Bootstrap
```

Vamos a usarlo junto con una librería complementaria `Flask-Bootstrap` que lo único que hace es generar el código de HTML del formulario con loa esctuctura y las clases de [Twitter Bootstrap](http://getbootstrap.com/2.3.2/) (lo mismo que hace `django-crispy-forms`).

{% codeblock users/forms.py lang:python %}
import wtforms
from flask_wtf import Form

class UserForm(Form):
    email = wtforms.StringField(
        validators=[
            wtforms.validators.Email(),
            wtforms.validators.DataRequired(),
        ],
    )
    username = wtforms.StringField(
        validators=[wtforms.validators.DataRequired()],
    )
    submit = wtforms.SubmitField('Save')
{% endcodeblock %}

Ahora podemos renderiizar el formulario en nuestra plantilla:

{% codeblock users/templates/users/form.html lang:html %}{% raw %}
{% extends "layouts/main_layout.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block content %}
  {{ wtf.quick_form(form) }}
{% endblock %}
{% endraw %}{% endcodeblock %}

Éste código nos va a renderizar el formulario completo con todos los campos especificados. Si queremos renderizar sólo algunos campor específicos, podemos escrebirlo de la siguiente forma:

{% codeblock users/templates/users/form.html lang:html %}{% raw %}
{{ wtf.form_field(form.email) }}
{{ wtf.form_field(form.submit) }}
{% endraw %}{% endcodeblock %}

Lo único que hace falta es escribir una vista que resiba los datos del formularioi y los guarde en la base de datos, por ejemplo.

El diccionario con los valores para cada campo se encuentra en la variable `request.form`. Si el request tiene método POST, validamos el formulario `form.validate()`, y en el caso exitoso, pasamos los valores del formulario a nuestro objeto: `form.populate_obj(user)`.

{% codeblock users/views.py lang:python %}
@users.route('/update/<int:pk>/', methods=('GET', 'POST'))
def user_update(pk):
    user = User.query.filter_by(id=pk).first_or_404()

    form = UserForm(user, request.form)

    if request.method == 'POST' and form.validate():
        form.populate_obj(user)

        db.session.add(user)
        db.session.commit()

        flash('Usuario fue editado exitosamente', 'success')
        return redirect(url_for('users.user_detail', pk=user.id))

    return render_template(
        'users/form.html',
        form=form,
    )
{% endcodeblock %}

## Autenticación

Para agregar autenticación al proyecto de Flask, normalmente usan dos siguentes librerías:

```
$ pip install Flask-Login
$ pip install Flask-OAuth
```

La primera tiene toda la funcionalidad de logeo, logout de usuario. La podemos instalar de la misma forma, que hicimos con SQLAlchemy hace poco:

{% codeblock run.py lang:python %}
from flask_login import LoginManager

login_manager = LoginManager()
login_manager.login_view = 'users.login'

def create_app():
    app = Flask(__name__, static_url_path='/static')
    app.config.from_object('conf.config')
    app.permanent_session_lifetime = datetime.timedelta(days=365)

    login_manager.init_app(app)
{% endcodeblock %}

En `login_manager.login_view` especificamos qué vista corresponde a login, y en `permanent_session_lifetime`  se puede indicar el tiempo en el qué expirará la sesión.

El modelo que vamos a usar para usuarios debe heredar de `UserMixin` de `flask_logins`

{% codeblock users/models.py lang:python %}
from flask_login import UserMixin
from run import db

class User(db.Model, UserMixin):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    # ...
{% endcodeblock %}

Y ahora usando la otra librería - `Flask-OAuth` - podemos hacer una vista para nuestros usuarios puedan hacer login desde su cuenta de Google, por ejemplo.

{% codeblock users/views.py lang:python %}
@users.route('/login')
def login():
    callback = url_for('users.authorized', _external=True)
    return google.authorize(callback=callback)
{% endcodeblock %}

Cuando el usuario ya está autenticado, en cualquer punto de nuestra aplicación podemos preguntar por el objero correspondiente: 

```python
from flask_login import current_user
```

Más información acerca de cómo congigurar los tokens de Google se puede encontrar en la documentación de la librería: [pythonhosted.org/Flask-OAuth/](https://pythonhosted.org/Flask-OAuth/).

## Múltiples idiomas

Si queremos tener soporte de multiples idiomas, tendremos que instalar otra librería:

```
$ pip install Flask-Babel
```

Mirarmos qué comandos nos ofrece:

{% codeblock lang:bash %}
# Extraer los textos para traducción 
# (se corre sólo una vez al principio)
$ pybabel extract -F babel.cfg -o messages.pot .


# Generar un catálogo para español
$ pybabel init -i messages.pot -d translations -l es

# Se crea el directorio translations/es
# Por dentro hay otro directorio llamado LC_MESSAGES que tiene  
# un archivo messages.po. 

# Después de traducir los textos y guardarlos en messages.po,
# compilamos el archivo y publicamos los textos:

$ pybabel compile -d translations

# Para actualizar las traducciones a diario:

$ pybabel extract -F babel.cfg -o messages.pot .
$ pybabel update -i messages.pot -d translations
{% endcodeblock %}

Si queremos traducir cadenas de texto dentro del código de Python, habrá que usar la siguiente sintaxis:

{% codeblock lang:python %}
from flask_babel import gettext as _

_('Invalid authentication token')
{% endcodeblock %}

Y en plantillas se ve muy parecido:

{% codeblock lang:html %}{% raw %}
<button class="btn btn-default" title="{{ _('Help') }}">
    <i class="fa fa-question"></i>
</button>
{% endraw %}{% endcodeblock %}

## Mensajes

Los que están familiares con el framework Django, recuerden que Django tiene un procesador de contexto `messages` que nos permite mandar mensajes a las plantillas desde el código Python:

{% codeblock lang:python %}
from django.contrib import messages

messages.add_message(request, messages.INFO, 'Hello world.')
{% endcodeblock %}

En Flask es muy parecido, sino esos mensajes se llaman `flash`:

{% codeblock lang:python %}
from flask import flash

flash('Hello world.', 'success')
{% endcodeblock %}

y para consultarnos dentro de la plantilla llamamos el método `get_flashed_messages`:

{% codeblock lang:html %}{% raw %}
{% for category, message in get_flashed_messages(with_categories=true) %}
<div class="alert alert-{{ category|replace('message', 'info') }}">
  <button type="button" class="close" data-dismiss="alert">×</button>
  {{ message }}
</div>
{% endfor %}
{% endraw %}{% endcodeblock %}

## Cache

Y para terminar la presentación, vamos a meter todo en cache #comonosgusta:

```
$ pip install Flask-Cache
```

Ya conocemos cómo instalar las aplicaciones externas en Flask, pero lo repasamos:

{% codeblock run.py lang:python %}
from flask_cache import Cache


cache = Cache()

def create_app():
    app = Flask(__name__, static_url_path='/static')
    app.config.from_object('conf.config')

    cache.init_app(app)
{% endcodeblock %}

Y ahora sí, vamos con toda... ponemos en caché una vista completa:

{% codeblock users/views.py lang:python %}
from run import cache

@users.route('/list')
@cache.cached(timeout=60*60, key_prefix='user_list')
def user_list():
    pass
{% endcodeblock %}

y luego un fragmento de html:

{% codeblock lang:html %}{% raw %}
{% cache 60*60, 'dashboard_menu_user' + current_user.id|string %}
  {% include 'layouts/_menu.html' %}
{% endcache %}
{% endraw %}{% endcodeblock %}

## pip freeze

Y ahora vamos resumir qué librerías hemos instalado y cuals otros nos podrías ser útil en el futuro:

{% codeblock %}{% raw %}
# Framework
Flask==0.11

# Libs
celery==3.1.20
coverage==4.0.3
factory-boy==2.6.1
SQLAlchemy==1.0.12
SQLAlchemy-Utils==0.31.6

# Flask libs
Flask-And-Redis==0.6
Flask-Babel==0.9               # Múltiples idiomas y zonas horarias
Flask-Bootstrap==3.3.5.7       # ~ django-crispy-forms para Bootstrap
Flask-Cache==0.13.1
Flask-DebugToolbar==0.10.0     # En Django: django-debug-toolbar
Flask-fillin==0.2              # Diligenciar formularios en pruebas
Flask-Login==0.3.2
Flask-Migrate==1.8.0
Flask-Moment==0.5.1            # Integración con moment.js
Flask-OAuth==0.12              # ~ python-social-auth
Flask-Script==2.0.5            # manage.py, shell
Flask-SQLAlchemy==2.1
Flask-WTF==0.12                # Formularios con protección CSRF
WTForms-Alchemy==0.15.0        # Para crear formularios basados en modelos
WTForms-Components==0.10.0     # Campos adicionales para los formularios de Flask
{% endraw %}{% endcodeblock %}

{% img center /images/flask/nerd-dad.jpg %}

Gracias por su atención y a [Tappsi](https://tappsi.co/) por al apoyo.

{% img center /images/flask/tappsi_logo.svg 100 %}

Estamos contratando jobs@tappsi.co