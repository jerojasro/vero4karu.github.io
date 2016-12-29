---
layout: post
title: "Monitoreo de acciones de usuarios en Flask (SQLAlchemy)"
date: 2016-07-22 08:23:58 -0500
comments: true
categories: 
- Flask
- SQLAlchemy
---

Suponemos que en nuestro proyecto de Flask hay un modelo `Client` definidao usando [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/) y queremos monetorear los cambios que se realizan sobre los objetos de ese modelo.

Para esto agregamos tres señales:

```python models.py
from sqlalchemy import event

from main.signals import receive_before_update, receive_before_insert, receive_before_delete

class Client(db.Model):
    id = Column(Integer, primary_key=True)
    name = Column(String(255))
    email = Column(String(255), unique=True)
    phone = Column(String(255))

event.listen(Client, 'before_insert', receive_before_insert)
event.listen(Client, 'before_update', receive_before_update)
event.listen(Client, 'before_delete', receive_before_delete)
```

EN el gódigo de cada señal llamamos los métodos `inspect()` y `get_history()` para detectar los cambios sobre el objeto:

```python
state = db.inspect(client_object)
for attr in state.attrs:
     hist = state.get_history(attr.key, True)
```

Por ejemplo, para el atributo `name`:

```json
{
    'added': ['Company A'],
    'deleted': ['Company B'],
}
```

El mégoto `get_history()` nos devuelve un objeto `sqlalchemy.orm.attributes.History` que tiene los siguientes atributos:

* `added` - listado de valores agregados al atributo de nuestro objeto
* `deleted` - listado de valores eliminados del atributo de nuestro objeto
* `unchanged` - listado de valores del atributo de nuestro objeto que se quedaron intactos
* `has_changes()` - método que retorna `True` si no habían ningunos cambios de valor de nuestro atributo.

Ahora, usando la variable `current_user` de la biblioteca [Flask-Login](https://flask-login.readthedocs.io/), podemos guargar el usuario que realizó los cambios:

```python signals.py
from flask_login import current_user

def save_changes(target, action):
    try:
        state = db.inspect(target)
        changes = {}
        for attr in state.attrs:
            hist = state.get_history(attr.key, True)
            if not hist.has_changes():
                continue
            added = format_changes_value(hist.added)
            deleted = format_changes_value(hist.deleted)
            if added != deleted:
                changes[attr.key] = {
                    'added': added,
                    'deleted': deleted,
                }
        if changes:
            mongo.db.user_logs.insert_one(dict(
                current_user_id=current_user.id,
                action=action,
                table_name=target.__tablename__,
                object_id=target.id,
                changes=changes,
                created_at=datetime.datetime.utcnow(),
            ))
    except Exception as e:
        pass

def receive_before_insert(mapper, connection, target):
    save_changes(target, 'create')


def receive_before_update(mapper, connection, target):
    save_changes(target, 'update')


def receive_before_delete(mapper, connection, target):
    save_changes(target, 'delete')
```
