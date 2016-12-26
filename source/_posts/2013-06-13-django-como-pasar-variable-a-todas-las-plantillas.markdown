---
layout: post
title: "Django: como pasar variable a todas las plantillas"
date: 2013-06-13 19:11:19 -0500
comments: true
categories:
---

Primero hay que escribir su propio procesador de contexto:

```python
TEMPLATE_CONTEXT_PROCESSORS = (
    'django.core.context_processors.debug',
    'django.core.context_processors.i18n',
    'django.core.context_processors.media',
    'django.core.context_processors.static',
    'django.contrib.auth.context_processors.auth',
    'django.contrib.messages.context_processors.messages',
    'django.core.context_processors.request',

    'miproyecto.miapp.context_processors.hola',
)
```

En `context_processors.hola.py`:

```python
from django.core.context_processors import request

def hola(request):
    return {'hola_mundo': 'Hola mundo!'}
```

Y en la plantilla:

{% codeblock lang:html %}
{% raw %}
{{ hola_mundo }}
{% endraw %}
{% endcodeblock %}
