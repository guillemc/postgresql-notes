Query Examples
==============




### Inserts


#### Insert / update on key error

This would be the equivalent for Mysql's `insert on duplicate ket update`:

```
INSERT INTO rewrites (source, destination, created_at, updated_at)
VALUES (?, ?, ?, ?)
ON CONFLICT (source) DO UPDATE SET destination = excluded.destination, updated_at = excluded.updated_at
```
(`source` is a column with an unique index)


### Updates
#### Update multiple rows with different values

```
UPDATE users AS u SET
  email = u2.email,
  name = u2.name  

FROM (VALUES
  (1, 'hollis@weimann.biz', 'Hollis O\'Connell'),
  (2, 'robert@duncan.info', 'Robert Duncan')
) AS u2(id, email, name)

WHERE u2.id = u.id;

```
