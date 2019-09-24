---
title: Complex Data Types
description: |
  A set of native datatypes that allow manipulation of Lists, Maps and Geospatial data.  
---

Aerospike supports a rich set of complex data types
  - List
  - Map
  - Geospatial

Each of these datatypes has specific built-in operators that allow for manipulation and query of these dataypes in a natural way.

### [List](/docs/guide/cdt-list.html)

List is an ordered collections of values. A list may contain values of any of the supported data-types. The data in the list is insert ordered, and the order is maintained when stored and retrieved. See [Complex Data Types - List](/docs/guide/cdt-list.html) for additional information, including operators that allow manipulation of the List directly on the Server.

```bash
aql> insert into test.demo (PK, bin) values ('key', 'JSON[1,2,3]')
OK, 1 record affected.

aql> select * from test.demo where PK='key'
+-----------+
| bin       |
+-----------+
| [1, 2, 3] |
+-----------+
1 row in set (0.000 secs)
```

### [Map](/docs/guide/cdt-map.html)

Map is a collection of key-value pairs, such that each key may only appear once in the collection and is associated with a value. The key and value of a map may be of any of the supported data-types. See [Complex Data Types - Map](/docs/guide/cdt-map.html) for additional information.

```bash
aql> insert into test.demo (PK, bin) values ('key', 'JSON["str1" , "str2" , "str3"]')
OK, 1 record affected.

aql> select * from test.demo where PK='key'
+--------------------------+
| bin                      |
+--------------------------+
| ["str1", "str2", "str3"] |
+--------------------------+
1 row in set (0.000 secs)
```


List and maps allow arbitratry nesting of one into another. Using which arbitrary document can be created

```bash
aql> insert into test.demo (PK, bin) values ('key', 'JSON["string", 10, ["list" ,"of" , "strings"], {"map":1, "of":2, "items":3}]')
OK, 1 record affected.

aql> select * from test.demo where PK='key'
+-------------------------------------------------------------------------+
| bin                                                                     |
+-------------------------------------------------------------------------+
| ["string", 10, ["list", "of", "strings"], {"items":3, "of":2, "map":1}] |
+-------------------------------------------------------------------------+
1 row in set (0.000 secs)
```

The JSON document passed to `aql` is converted into list with a string, a integer, a list, a map value in it

### [Geospatial](/docs/guide/geospatial.html)
- *GeoJSON format* for specifying and storing object geometry, achieving data standardization and interoperability. See [GeoJSON Specification 1.0](http://geojson.org/geojson-spec.html) and [GeoJSON IETF WG](https://github.com/geojson/draft-geojson).
- *S2 Spherical Geometry Library* for mapping points and regions into single dimension 64bit CellID representation. Details on S2, Hilbert Curve, and Geospatial mapping is best explained in this [S2 Blog](http://blog.christianperone.com/2015/08/googles-s2-geometry-on-the-sphere-cells-and-hilbert-curve/).
- Aerospike's existing *Secondary Index and Query* capabilty to achieve peformance and scale on index insert/update and query. More details on [Secondary Index Architecture](/docs/architecture/secondary-index.html) and [Query Guide](/docs/guide/query.html).

For example, to search for points with a Region:

```python
def query_pwr(args,client):

    """Construct a GeoJSON region."""
    region = aerospike.GeoJSON({
        'type': 'Polygon', 
        'coordinates': [[[-122.500000,37.000000],
                         [-121.000000, 37.000000], 
                         [-121.000000, 38.080000],
                         [-122.500000, 38.080000], 
                         [-122.500000, 37.000000]]]})

    """Construct the query predicate."""
    query = client.query(args.nspace, args.set)
    query.where(aerospike.predicates.geo_within_geojson_region(LOCBIN, region.dumps()))
    
    """Define callback to process query result."""
    def callback((key, metadata, record)):
        records.append(record)

    """Make the actual query!"""

    query.foreach(callback)

```
