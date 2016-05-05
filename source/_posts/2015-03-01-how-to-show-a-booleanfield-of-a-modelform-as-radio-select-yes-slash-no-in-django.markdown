---
layout: post
title: "How to show a BooleanField of a ModelForm as radio select (yes/no) in Django"
date: 2015-03-01 19:26:52 -0500
comments: true
categories: [Django]
---

Let's suppose that we have a field ``is_active`` in our model. It is a boolean field, but in a template we want to show it as a radio select.

In this case we just need to add ``choices`` attribute for the model field and then change a widget of the corresponding form:

{% gist ff27cad71d8f7f80f735 %}
