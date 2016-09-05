---
layout: post
title: "How to show a BooleanField of a ModelForm as radio select (yes/no) in Django"
date: 2015-03-01 19:26:52 -0500
comments: true
categories: [Django]
---

Let's suppose that we have a field ``is_active`` in our model. It is a boolean field, but in a template we want to show it as a radio select.

In this case we just need to add ``choices`` attribute for the model field and then change a widget of the corresponding form:

```python forms.py
class MyForm(forms.ModelForm):
    class Meta:
        model = MyModel
        widgets = {
            'is_active': forms.RadioSelect
        }
```

```python forms.py
from django.db import models

class MyModel(models.Model):
    STATE_CHOICES = (
        (True, u'Yes'),
        (False, u'No'),
    )
    is_active = models.BooleanField(
        verbose_name=u'Is it active?',
        default=False,
        choices=STATE_CHOICES,
    )
```

