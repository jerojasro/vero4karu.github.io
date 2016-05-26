---
layout: post
title: "Cómo obtener el nombre del día de la semana en Python"
date: 2016-05-26 09:02:36 -0500
comments: true
categories: Python, datetime
---

```python
> import calendar
> calendar.day_name[0]
'Monday'
> calendar.day_name[1]
'Tuesday'
> calendar.day_name[2]
'Wednesday'
> calendar.day_name[3]
'Thursday'
> calendar.day_name[4]
'Friday'
> calendar.day_name[5]
'Saturday'
> calendar.day_name[6]
'Sunday'
```

Si para nosotros la semana empieza el Domindo, podemos escribirlo de la siguiente forma:

```python
> sunday = 0
> calendar.day_name[sunday - 1 % 7]
'Sunday'

> tuesday = 2
> calendar.day_name[tuesday - 1 % 7]
'Tuesday'
```