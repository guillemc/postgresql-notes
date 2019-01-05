Jsonb queries
==============

### Setting raw json

```
UPDATE banners SET config = '{}' WHERE id = 1;
UPDATE banners SET config = '{"country": "ES"}' WHERE id = 1;
```

Merging values:

```
UPDATE banners SET config = config || '{"size": [300, 200]}' WHERE id = 1;
UPDATE banners SET config = config || '{"tags": ["sports", "tv"]}' WHERE id = 1;
```

(the current value must not be NULL for this to work)

Removing a key:

```
UPDATE banners SET config = config - 'tags' ;
```

### Getting values

```
SELECT config->'country' AS country FROM banners WHERE id = 1;
SELECT config->'size' AS size FROM banners WHERE id = 1;
SELECT config->'size'->1 AS height FROM banners WHERE id = 1;
```

### Searching for values

The `->` operator returns a jsonb object. The `->>` operator returns a text value.

```
SELECT * FROM modules where content->>'image_id' = '38761';  
```

If we want to compare a path to an int, for example, we can use conversion functions:
```
SELECT * FROM modules where content->'image_id' = to_jsonb(38761);
SELECT * FROM modules where (content->>'image_id')::int = 38761;
```

Or use the containment operator `@>`:

```
SELECT * FROM modules WHERE content @> '{"image_id": 38761}';
```

Using the containment operator we can also search for elements inside arrays:

```
SELECT * FROM banners WHERE config @> '{"tags": ["tv"]}';
```


### Checking key existence / absence

```
SELECT * FROM banners WHERE config->'size' IS NOT NULL;
SELECT * FROM banners WHERE config->'whatever' IS NULL;
```

Note that this is different from checking for a null value inside the json:

```
SELECT * FROM banners WHERE config @> '{"whatever": null}';
```

To check for both cases (key doesn't exist or has a null value):

```
SELECT * FROM banners WHERE config->>'whatever' IS NULL;
```



### Creating indexes on jsonb fields

For searches on jsonb fields with the `=` we can use a B-tree index (this is the default index type):

```
CREATE INDEX idx_banners_config_country ON banners ((config->>'country'));

SELECT * FROM banners WHERE config->>'country' = 'ES';
```

For searches with the containment operator we can use a GIN index:
```
CREATE INDEX idx_banners_config_tags ON banners USING GIN (config);

SELECT * FROM banners WHERE config @> '{"country": "ES"}'
```

See also [B-tree vs GIN](https://bitnine.net/blog-postgresql/postgresql-internals-jsonb-type-and-its-indexes/)

### TO DO

json_array_elements, jsonb_array_elements_text
