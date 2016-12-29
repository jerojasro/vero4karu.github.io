---
layout: post
title: "Cómo pasar variable desde una vista a formulario en Django"
date: 2016-10-21 14:07:56 -0500
comments: true
categories: 
- Django
- Python
---

```python views.py
class MyCreateView(CreateView):
    model = MyModel
    template_name = 'myapp/my_form.html'
    form_class = MyForm
    
    def get_form_kwargs(self):
        kwargs = super(MyCreateView, self).get_form_kwargs()

        # La variable que queremos pasar al formulario
        kwargs.update({'current_user': self.request.user})

        return kwargs
```

```python forms.py
class MyForm(forms.ModelForm):
    
    def __init__(self, *args, **kwargs):
        # Recibir la variable y borrarla del listado de argumentos.
        current_user = kwargs.pop('current_user')

        super(MyForm, self).__init__(*args, **kwargs)
```


