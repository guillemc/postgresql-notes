Query Examples
==============

### Selects

#### Mysql's group_concat equivalent

We can use `array_agg`, which groups results into an array:

```
SELECT a.id, a.title, array_agg(at.tag) AS tags
FROM articles a LEFT JOIN article_tag at ON at.article_id = a.id
GROUP BY a.id
```

| id  | title       | tags |
| --- | ----------- | ---- |
| 36  | The National | {"indie rock",country-rock,folk-rock} |
| 37  | Hinds        | {NULL} |


The array can be ordered:
```
array_agg(at.tag ORDER BY tag)
```

And converted to a string:
```
array_to_string(array_agg(at.tag ORDER BY tag), ', ')
```

This can be done at once with `string_agg`:
```
string_agg(at.tag, ', ' ORDER BY tag)
```



### Inserts


#### Insert / update on key error

This would be the equivalent for Mysql's `insert on duplicate ket update`:

Example - table that stores redirections, where `source` is a column with an unique index:
```
INSERT INTO rewrites (source, destination, created_at, updated_at)
VALUES (?, ?, ?, ?)
ON CONFLICT (source) DO UPDATE SET destination = excluded.destination, updated_at = excluded.updated_at
```



### Updates
#### Update multiple rows with different values

Example - we receive a list tuples with (id, position) and update all rows at once:
```
UPDATE options AS tbl SET position = tmp.position FROM
(VALUES (113, 1), (115, 2), (114, 3), (112, 5)) AS tmp(id, position)
WHERE tbl.id = tmp.id
```

Example - update names and emails on a users table:
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
