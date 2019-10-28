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

If user needs to have superuser privileges:
```
ALTER USER tom WITH SUPERUSER;
ALTER USER tom WITH NOSUPERUSER;
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


### Export a table or query to csv

```
COPY (select * from properties_export) TO '/tmp/export.csv' WITH (FORMAT csv, HEADER)
COPY (select * from properties_export) TO PROGRAM 'gzip > /tmp/export.csv.gzip' WITH (FORMAT csv, HEADER, DELIMITER ';')
```
The copy command requires the user to be superuser. 

There's also the equivalent psql \copy meta-command:

```
\copy (select * from properties_export) TO '/tmp/export.csv' WITH (FORMAT csv, HEADER)
```

### Checking installed version

We can do `psql --version` or execute the query `select version();`

### Table maintenance

Restarting sequences:

```
ALTER SEQUENCE users_id_seq RESTART WITH 1;
```

Plain vacuum frees up space:

```
VACUUM mytable;
```

Full vacuum can reclaim more space, but locks the table and takes longer:

```
VACUUM(FULL) mytable;
```

Full vacuum + analyze will also update stats used for query efficiency:

```
VACUUM(FULL, ANALYZE) mytable;
```
### Tables

```
CREATE TABLE IF NOT EXISTS street_names (
  id SERIAL PRIMARY KEY,
  gml_id VARCHAR(30) NOT NULL UNIQUE,
  name VARCHAR,
  code CHARACTER(5) NOT NULL
);
```

### Schemas

```
CREATE SCHEMA upload;

GRANT ALL on schema upload to testgis;

ALTER TABLE upload.cities SET SCHEMA public;
```

Set the search_path at runtime:
```
SET search_path = upload;
````

Permanently set the search path for a user:
```
ALTER USER admin SET search_path = upload, public;
```

### Killing long queries

```
select pid, usename, query from pg_stat_activity where state = 'active';

-- take note of the pid, for example: 29458

select pg_terminate_backend(29458);
```
