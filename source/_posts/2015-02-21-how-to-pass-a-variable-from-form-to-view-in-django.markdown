---
layout: post
title: "How to pass a variable from form to view in Django"
date: 2015-02-21 11:09:20 -0500
comments: true
categories: [Django]
---

Sometimes there is a need to pass a variable (parameter) from a ``FormView`` (for example, ``CreateView``,  ``UpdateView``) to a form (for example, ``ModelForm``).

Let's suppose that we want to use a value of ``other_variable`` from a view ``MyCreateView`` in ``MyForm``:

{% gist ec0f82bb3d302961503d %}