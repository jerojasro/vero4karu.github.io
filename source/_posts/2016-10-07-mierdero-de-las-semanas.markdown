---
layout: post
title: "Mierdero de las semanas"
date: 2016-10-07 11:53:05 -0500
comments: true
categories: 
---



Bienvenidos al mierdero de las semanas en postgresql, mysql y Python:

Día  | postgresql: `EXTRACT(DOW...)` | Python: `isoweekday()` | Python: `weekday()` | mysql: `DAYOFWEEK(...)`
:------------|:-------|-------------|-------------|----------
Sun | 0 | 7 | 6 | 1
Mon | 1 | 1 | 0 | 2
Tue | 2 | 2 | 1 | 3
Wed | 3 | 3 | 2 | 4
Thu | 4 | 4 | 3 | 5
Fri | 5 | 5 | 4 | 6
Sat | 6 | 6 | 5 | 7

**Java**: desde 1 (Monday) hasta 7 (Sunday) ([docs](https://docs.oracle.com/javase/8/docs/api/java/time/DayOfWeek.html))

Enlaces:

* [mysql: function dayofweek](https://dev.mysql.com/doc/refman/5.5/en/date-and-time-functions.html#function_dayofweek)
* [postgresql: functions datetime](https://www.postgresql.org/docs/current/static/functions-datetime.html)