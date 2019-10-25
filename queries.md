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
array_agg(at.tag ORDER BY tag) AS tags
```

And converted to a string:
```
array_to_string(array_agg(at.tag ORDER BY tag), ', ') AS tags
```

This can be done at once with `string_agg`:
```
string_agg(at.tag, ', ' ORDER BY tag) AS tags
```

In our client code most likely we'll receive arrays as strings.
When we need to work with an array in our client code, we can use `array_to_json`:
```
array_to_json(array_agg(at.tag ORDER BY tag)) AS tags
```
(Then, in our php client code, for example, we'd do `$tags = json_decode($row->tags)`)

#### Dealing with nulls

The `coalesce` function returns the first value that is not null:
```
SELECT id, coalesce(short_title, title) AS title FROM articles ORDER BY coalesce(short_title, title)
```

The `nullif(v1, v2)` function returns null if v1 equals v2 (and v1 otherwise).

We can use it to convert empty strings to null:
```
SELECT id, coalesce(nullif(short_title, ''), title) AS title FROM articles
```


#### Ordering by cases

The simple form is `CASE expression WHEN value THEN result [WHEN... THEN...] ELSE default_result END`:
```
SELECT id, name FROM countries ORDER BY CASE code WHEN 'ES' THEN 1 WHEN 'PT' THEN 1 ELSE 2 END, name
```

The general form is `CASE WHEN condition THEN result [WHEN... THEN...] ELSE default_result END`:
```
SELECT id, name FROM countries ORDER BY CASE WHEN code = ANY('{"ES", "PT"}') THEN 1 ELSE 2 END, name
```


### Inserts


#### Insert / update on key error

This would be the equivalent for Mysql's `insert on duplicate key update`.

Simplest case - ignore conflicts:
```
INSERT INTO street_names (id, name) SELECT gml_id, address_text FROM addresses ON CONFLICT DO NOTHING
```

Example - table that stores redirections, where `source` is a column with an unique index:
```
INSERT INTO rewrites (source, destination, created_at, updated_at)
VALUES (?, ?, ?, ?)
ON CONFLICT (source) DO UPDATE SET destination = excluded.destination, updated_at = excluded.updated_at
```

In case the constraint is on a combination of fields, we'll use the constraint's name:
```
INSERT INTO translations (source, language, destination, created_at, updated_at)
VALUES (?, ?, ?, ?, ?)
ON CONFLICT ON CONSTRAINT translations_source_language_unique DO UPDATE SET updated_at = excluded.updated_at
```


### Updates

#### Update with join

Example - uploads table with duplicate paths, for each path we want to keep the one with the highest id and mark the others for deletion:

```
UPDATE uploads u SET is_temp = true 
FROM uploads u2    
WHERE
    u.path = u2.path    
    AND u.id < u2.id
```

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


#### Regular expressions

```
UPDATE docs SET filepath = REGEXP_REPLACE(filepath, '^(.*?)documents\\(.*?)\\(.*?)(\\.*?\.pdf)?$','\2/\3');
```

SIMILAR TO is more powerful than LIKE, and simpler than regular expressions:
```
SELECT table_name FROM information_schema.tables WHERE table_schema = 'upload' AND table_name SIMILAR TO '[0-9]{5}' ORDER BY table_name;
```
