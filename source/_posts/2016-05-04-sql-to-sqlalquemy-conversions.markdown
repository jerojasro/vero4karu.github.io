---
layout: post
title: "SQL to SQLAlquemy conversions"
date: 2016-05-04 21:15:39 -0500
comments: true
categories: [SQL, SQLAlquemy]
---

Some examples on how to convert raw SQL to SQLAlchemy query:

### Select all

```sql
SELECT COUNT(*)
```

```python
> from sqlalchemy import text, func
> db.session.query(func.count()).all()
```

### Add labels

```sql
SELECT COUNT(*) as requiests
```
```python
> rows = db.session.query(func.count().label('requests')).all()
> row = rows[0]
> row.requests
12
```

### Sum

```sql
SELECT SUM(status) as rides
```
```python
> rows = db.session.query(func.sum(Book.status).label('books')).all()
> row = rows[0]
> row.books
12
```

### Return a count of rows and distinct

```sql
SELECT COUNT(DISTINCT(author_id)) as authors
```

```python
from sqlalchemy import distinct
> rows = db.session.query(
    func.count(distinct(Book.author_id)).label('authors')
).all()
```

### Query a date range

```sql
SELECT * FROM b WHERE b.created_at > current_timestamp - (current_timestamp - interval '5 hours')::time
```
```python
> now = datetime.datetime.utcnow()
> db.session.query(Book).filter(
    Book.created_at.between(now - datetime.timedelta(hours=5), now)
).all()
```

### Conditional sum

```sql
SELECT SUM(((status IN (4, 7))::int)) as books ...
```
```python
> from sqlalchemy.sql.expression import case
> db.session.query(
    func.sum(case([(Book.status.in_((4, 7)), 1)], else_=0)).label('books'),
).filter(
    # ...
).all()
```

### In

```python
> Book.query.filter(
    Book.status.in_((
        BOOK_CONFIRMED,
        BOOK_FINISHED,
    )),
).count()
```

### Other examples

```python
bookings = db.session.query(
    func.to_char(Book.created_at, 'MM'),
    func.count(Book.created_at)
).filter(
    Book.author_id == author.id,
).filter(
    Book.created_at.between(six_months_ago_min, today_max)
).group_by(
    func.to_char(Book.created_at, 'MM')
).order_by(func.to_char(Book.created_at, 'MM')).all()
```