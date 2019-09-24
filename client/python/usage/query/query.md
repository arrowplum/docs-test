---
title: Query Records
description: Use the Aerospike Python client APIs to query the database using secondary indexes.
---

Use the Aerospike Python client APIs to query the database using secondary indexes.

### Creating a Query

`client.query()` takes the `namespace` (required) and `set` (optional) arguments. `set` can be ommited or `None`. The return value is a new `aerospike.Query` class instance.

This example creates a query on the *test* namespace, *demo* set:

```python
query = client.query('test', 'demo')
```

### Projecting Bins

Project bins using `select()` on the `Query` class instance. `select()` accepts one or many bin names (strings).

This example selects *name* and *age* bins from the specified records:

```python
query.select('name', 'age')
```

### Adding Query Predicates

Define predicates using `where()` on the `Query` class instance. `where()` accepts a predicate created using one of the functions in `aerospike.predicates`, including:

- `equals(bin, value)` &mdash; Find records containing bin *bin* with the specified *value* (integer or string).
- `between(bin, min, max)` &mdash; Find records containing bin *bin* with a value in the *min* and *max* range (integer only).

Use `p` to import the predicates module from `aerospike.predicates`.

This example adds a `between()` predicate to a query:

```python
query.where( p.between('age', 14, 25) )
```

### Reading Results

Execute the query and read the results using `foreach()` in the `Query` class instance. `foreach()` accepts a callback function for each result read from the query. The callback function must accept a single argument as a tuple:

- `key tuple` &mdash; The tuple to identify the record.
- `metadata` &mdash; The dict containing the record metadata (TTL and generation).
- `record` &mdash; The dict containing the record bins.

If the callback returns `False`, the client stops reading results.

These examples execute the query and reads results.

To print the records as they are read:

```python
def print_result((key, metadata, record)):
    print(key, metadata, record)
```

To execute the query and call `print_result` for each result:

```python
query.foreach(print_result)
```

