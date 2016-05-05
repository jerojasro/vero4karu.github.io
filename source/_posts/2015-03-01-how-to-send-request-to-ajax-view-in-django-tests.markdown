---
layout: post
title: "How to send request to AJAX view in Django tests"
date: 2015-03-01 19:23:37 -0500
comments: true
categories: [Django, unittests, AJAX]
---

If you have a view that requires an AJAX request, in other words, checks if ``request.is_ajax()``, here is a way you can write a unit test for this view:

{% gist dca59e325c9f7554d176 %}