psql
====

In Ubuntu, first `sudo su postgres`, then execute `psql`

List databases: `\l`

Select database: `\c mydb`

List tables of the current database: `\dt`

Describe table: `\d articles`

### Creating databases

```
CREATE USER tom WITH PASSWORD 'jerry';
CREATE DATABASE mydb;
GRANT ALL PRIVILEGES ON DATABASE mydb TO tom;
```

Then install needed extensions:
```
\c mydb
CREATE EXTENSION IF NOT EXISTS unaccent;
```

If databases are not being created in your preferred encoding (UTF8)
we need to change the encoding of the `template1` database:

```
UPDATE pg_database SET datistemplate = FALSE WHERE datname = 'template1';
DROP DATABASE template1;
CREATE DATABASE template1 WITH TEMPLATE = template0 ENCODING = 'UTF8';
UPDATE pg_database SET datistemplate = TRUE WHERE datname = 'template1';
\c template1
VACUUM FREEZE;
```

### Renaming databases

```
ALTER DATABASE mydb RENAME TO newdb;
```


### Exporting and importing databases

```
$ pg_dump mydb > mydb.sql
$ psql mynewdb < mydb.sql  
```

Note: `mynewdb` must be an empty database.
