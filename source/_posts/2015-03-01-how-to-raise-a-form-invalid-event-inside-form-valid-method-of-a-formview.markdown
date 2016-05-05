---
layout: post
title: "How to raise a form invalid event inside form_valid method of a FormView"
date: 2015-03-01 19:35:52 -0500
comments: true
categories: [Django]
---

How to raise form invalid inside ``form_valid`` method of a ``FormView`` (``CreateView``/``UpdateView``) and add an error message to ``non_field_errors``:

{% gist 3b62a13bdce7fe4178ac %}