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

Empezamos con instalar Flask:

```
$ mkvirtualenv pycon2017 -p python3
$ pip install Flask
```

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

Corrimos la aplicación con el siguiente comando:

```
$ python app.py 
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

Y ahora lo podemos abrir en nuestro navegador:

{% img center /images/flask/flask_01.png %}



We're hiring [jobs@tappsi.co](mailto:jobs@tappsi.co)