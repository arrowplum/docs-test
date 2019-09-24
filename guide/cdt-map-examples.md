---
title: Map Examples
description: Examples of modeling with Aerospike maps
---

Aerospike [maps](/docs/guide/cdt-map.html) can be used to implement use cases
such as:

- Event History Containers
- Document Store
- Leaderboards

## Modeling Concepts

Nested lists and maps may be combined to model complex use cases.
In the examples below, the pseudo code is described in the
[map operations](/docs/guide/cdt-map-ops.html). Each language client for
Aerospike, such as Java or Python, will have equivalent methods or functions
to these pseudocode ones.

{{#note}}
Aerospike supports multi-operation transactions, which execute in order on one or
more bins of a single record (under a record lock).
In all language clients such a transaction is invoked using the **operate()**
method.
{{/note}}

{{#note}}
Starting with Aerospike version 4.6.0, list and map operations can be applied
to [deeply nested structures](/docs/guide/cdt-context.html).
{{/note}}

Developers familiar with other NoSQL document stores will see the flexibility in
applying map and list operations on bins with nested CDTs. Combined with
(operate) single record transactions, Aerospike provides powerful functionality for modeling
a wide range of use cases.

## Examples

### Event Containers with Unique Timestamps

In this example we want to store and query user event data. Each record contains
the recent N events of a specific user, keyed by that user's unique identifier.

Assuming that events will not occur at the same millisecond, we'll use millisecond timestamps as map keys for distinct events.
Each event's data will be a tuple `[ event-type, { attr1: v1, attr2: v2, ... } ]`.

Our sample data will be the following events of a single user:

```javascript
{
  1523474230000: ['fav',     {'sku':1, 'b':2}],
  1523474231001: ['comment', {'sku':2, 'b':22}],
  1523474236006: ['viewed',  {'foo':'bar', 'sku':3, 'zz':'top'}],
  1523474235005: ['comment', {'sku':1, 'c':1234}],
  1523474233003: ['viewed',  {'sku':3, 'z':26}],
  1523474234004: ['viewed',  {'sku':1, 'ff':'hhhl'}]
}
```

#### Retrieving data for specific event types
We'll retrieve all the events of a specific event type, using a [`get_all_by_value`](/docs/guide/cdt-map-ops.html#getAllByValue) map operation:

```javascript
get_all_by_value(['comment', *]) 
{
  1523474231001: ['comment', {'sku':2, 'b':22}],
  1523474235005: ['comment', {'sku':1, 'c':1234}]
}
```

The argument for the operation is the tuple `['comment', *]`. The [wildcard singleton](/docs/guide/cdt-ordering.html#wildcard)
(`*`) acts as a glob that matches _zero or more elements_. As [`get_all_by_value`](/docs/guide/cdt-map-ops.html#getAllByValue)
compares distinct values against each other (it is not a range operation) all
tuples starting with `'comment'` as their first element are matched.

{{#note}}
This modeling approach takes advantage of how Aerospike lists are compared to
each other. We can currently only rely on it to identify lists whose first
element is a specific value, followed by any number of 'trivial' attributes,
which we cannot query for by value.
{{/note}}

Expanding on that example, we will retrieve all the events for multiple event types:

```javascript
get_all_by_value_list([['comment', *], ['fav', *]])
{
  1523474230000: ['fav',     {'sku':1, 'b':2}],
  1523474231001: ['comment', {'sku':2, 'b':22}],
  1523474235005: ['comment', {'sku':1, 'c':1234}]
}
```

#### Counting Events

We will get a count of a specific event type against the sample data above by
specifying a [`resultType=count`](/docs/guide/cdt-map.html#map-result-types).
The default behavior would be that of [`resultType=KeyValue`](/docs/guide/cdt-map.html#map-result-types):

```javascript
get_all_by_value(['viewed', *], resultType=count) 
3
get_all_by_value(['comment', *], resultType=count) 
2
```

#### Trimming the Map

Often we want to cap the number of events captured within a map. We will use
the [`remove_by_index_range`](/docs/guide/cdt-map-ops.html#removeByIndexRange)
map operation to keep the last 1000 events:

```
remove_by_index_range(-1000, 1000, INVERTED, resultType=none)
```


### Event Containers with Unique UUIDS

In some use cases a timestamp would result in frequent collisions. We can model
using a unique identifier as the map key.

In this example a conversation thread is stored in a single record. The map keys
are message UUIDs, and the map values are list tuples `[timestamp, msg-string,
username]`.

Our sample data will be the messages in a single conversation thread:

```javascript
{
  '0edf5b73-535c-4be7-b653-c0513dc79fb4': [1523474230, "Billie Jean is not my lover", "MJ"],
  '29342a0b-e20f-4676-9ecf-dfdf02ef6683': [1523474241, "She's just a girl who", "MJ"],
  '31a8ba1b-8415-aab7-0ecc-56ee659f0a83': [1523474245, "claims that I am the one", "MJ"],
  '9f54b4f8-992e-427f-9fb3-e63348cd6ac9': [1523474249, "...", "Tito"],
  '1ae56b18-7a3c-4f64-adb7-2e845eb5094e': [1523474257, "But the kid is not my son", "MJ"],
  '08785e96-eb1b-4a74-a767-7b56e8f13ea9': [1523474306, "ok...", "Tito"],
  '319fa1a6-0640-4354-a426-10c4d3459f0a': [1523474316, "Hee-hee!", "MJ"]
}
```

We will retrieve all the messages in a range of timestamps, using the [`get_by_value_interval`](/docs/guide/cdt-map-ops.html#getByValueInterval) map operation:

```javascript
get_by_value_interval([1523474240, nil], [1523474246, nil])
{
  '29342a0b-e20f-4676-9ecf-dfdf02ef6683': [1523474241, "She's just a girl who", "MJ"],
  '31a8ba1b-8415-aab7-0ecc-56ee659f0a83': [1523474245, "claims that I am the one", "MJ"]
}
```

The arguments for the operation are minimum and maximum tuples of `[timestamp, nil]`. The [NIL singleton](/docs/guide/cdt-ordering.html) is lower in value than a string. The [`get_by_value_interval`](/docs/guide/cdt-map-ops.html#getByValueInterval) checks if each map value (a list) is between the two list arguments of the operation.

By the [interval comparison rules](/docs/guide/cdt-ordering.html#intervals) the
following is evaluated:

```bash
[1523474240, nil] ≰ [1523474230, "Billie Jean is not my lover", "MJ"] < [1523474246, nil] (false)
[1523474240, nil] ≤ [1523474241, "She's just a girl who", "MJ"]       < [1523474246, nil] (true)
[1523474240, nil] ≤ [1523474245, "claims that I am the one", "MJ"]    < [1523474246, nil] (true)
[1523474240, nil] ≤ [1523474249, "...", "Tito"]                       ≮ [1523474246, nil] (false)
[1523474240, nil] ≤ [1523474257, "But the kid is not my son", "MJ"]   ≮ [1523474246, nil] (false)
[1523474240, nil] ≤ [1523474306, "ok...", "Tito"]                     ≮ [1523474246, nil] (false)
[1523474240, nil] ≤ [1523474316, "Hee-hee!", "MJ"]                    ≮ [1523474246, nil] (false)
```

{{#note}}
Again, this modeling approach takes advantage of how Aerospike lists are compared to
each other. We can currently only rely on it to identify lists whose first
element is in a range of specified values. The subsequent 'trivial' attributes
cannot be queried by value.
{{/note}}

