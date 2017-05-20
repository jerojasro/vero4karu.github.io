---
layout: post
title: "Cheatsheet"
date: 2017-05-20 09:49:23 -0500
comments: true
categories: 
---

<!-- more -->

## PostgreSQL

* Installation: https://help.ubuntu.com/community/PostgreSQL
* Create a user: http://stackoverflow.com/questions/11919391/postgresql-error-fatal-role-username-does-not-exist
* Set password: http://stackoverflow.com/a/7696398/914332

#### Reset sequence

```sql
SELECT setval('mytable_id_seq', (SELECT MAX(id) FROM mytable));
```

#### Make a dump of existing database

```
$ pg_dump -O -U postgres database_name | gzip > /tmp/dump.sql.gz
```

#### Create dump without data

```
$ pg_dump test_db_development -s > /tmp/createdb.sql
```


#### Load database from dump

```
$ psql -U username -f /tmp/dump.sql.gz
$ psql -U username -d activar -f ~/Downloads/dump.sql

$ sudo -u postgres psql db_development < ~/Downloads/dump.sql
```

#### Duplicate database

```sql
CREATE DATABASE newdb WITH TEMPLATE olddb;
```

or

```
$ createdb -T olddb newdb
```

#### Download as csv

```sql
\COPY enterprises TO '/tmp/countries.csv' DELIMITER ';' CSV HEADER;
COPY (SELECT * FROM country WHERE country_name LIKE 'A%') TO '/usr1/proj/bray/sql/a_list_countries.copy';
```

#### Allow user to acces tables, functions and sequences

```sql
GRANT ALL ON ALL TABLES IN SCHEMA public TO user;
GRANT ALL ON ALL FUNCTIONS IN SCHEMA public TO user;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO user;

GRANT USAGE, SELECT ON SEQUENCE cities_id_seq TO user;
```

#### Create extension

```sql
CREATE EXTENSION unaccent;
CREATE EXTENSION hstore;
```

#### Set password for all users to 'demo'

```sql
UPDATE auth_user 
SET password ='bcrypt$$2a$12$udndek2c62VlLqWnAYU5qePYQ7SS9rmfnxIuGNhGR4EMfFadQsMuG';
```

#### Rename constraint 

```sql
ALTER TABLE name RENAME CONSTRAINT constraint_name TO new_constraint_name;
```

#### Create unique constraint

```sql
ALTER TABLE tablename ADD CONSTRAINT constraintname UNIQUE (columns);
```

#### Settings for query result tables

```
\pset expanded auto
\pset border 1
\pset pager on
\pset null '(NULL)'
```

#### Finding long running queries

```sql
SELECT
  pid,
  now() - pg_stat_activity.query_start AS duration,
  query,
  state
FROM pg_stat_activity
WHERE now() - pg_stat_activity.query_start > interval '5 minutes';
```

* [Finding and killing long running queries on PostgreSQL](https://medium.com/little-programming-joys/finding-and-killing-long-running-queries-on-postgres-7c4f0449e86d)

#### Cancel running queries 

```sql
SELECT pg_cancel_backend(__pid__);
```

## Heroku

* [Getting Started on Heroku with Python](https://devcenter.heroku.com/articles/getting-started-with-python)
* [Deploying Python and Django Apps on Heroku](https://devcenter.heroku.com/articles/deploying-python)

#### Login to Heroku from console

```
$ heroku login
```

#### Run Heroku app locally

```
$ heroku local web
```

#### Deploy your code

```
$ git push heroku master
```

#### View logs of your application

```
$ heroku logs --tail
```

#### Other commends

```
$ heroku run bash
$ heroku run python
$ heroku run python manage.py shell
$ exit
```