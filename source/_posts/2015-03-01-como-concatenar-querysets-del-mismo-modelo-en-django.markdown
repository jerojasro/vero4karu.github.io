---
layout: post
title: "¿Cómo concatenar querysets del mismo modelo en Django?"
date: 2015-03-01 19:21:21 -0500
comments: true
categories: [Django]
---

Modo #1: 

Podemos usar <code>itertools.chain</code> para  unir dos o más querysets:

    from itertools import chain
    for item in chain(qs1, qs2, qs3):
        # ...

Modo #2:

Podemos usar el operador lógico ``OR``:

    res = qs1 | qs2 | qs3
    res = res.distinct()
