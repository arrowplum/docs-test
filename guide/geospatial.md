---
title: Geospatial Index and Query
description: Use Aerospike geospatial storage, indexing, and query to enable fast queries on points within a region, on a region containing points, and points within a radius.
assets: /docs/guide/assets
---

Use Aerospike geospatial storage, indexing, and query to enable fast queries on points within a region, on a region containing points, and points within a radius.

The Aerospike geospatial feature relies on these technologies:

- **GeoJSON format** to specify and store [GeoJSON geometry objects](http://geojson.org/geojson-spec.html#geometry-objects), and achieve data standardization and interoperability.
 See the [GeoJSON Format Specification](http://geojson.org/geojson-spec.html) and [GeoJSON IETF WG](https://github.com/geojson/draft-geojson).
- The Aerospike [**AeroCircle**](#aerospike-geojson-extension) data type extends the GeoJSON format to store circles as well as polygons.
- **S2 Spherical Geometry Library** to map points and regions in single-dimension, 64-bit CellID representation.

{{#info}}
The [S2 Geometry Library](https://code.google.com/archive/p/s2-geometry-library/) is a spherical geometry library, very useful for manipulating regions on a sphere (commonly on Earth) and indexing geographic data. 
<br>
Also, review this [S2 blog](http://blog.christianperone.com/2015/08/googles-s2-geometry-on-the-sphere-cells-and-hilbert-curve/) for details on S2, Hilbert Curve, and Geospatial mapping.
{{/info}}

- Aerospike [secondary indexes](/docs/architecture/secondary-index.html) and [queries](/docs/guide/query.html) to achieve peformance and scale for index inserts/updates and queries.

## Use Cases 

- Vehicle tracking systems that require high throughput updates of vehicle location and frequently query vehicles within a region.
- Retail locations to find specific amenities within 500m of a shopper.
- Location-targetted bidding transactions to discover persons or devices within the location with an active ad campaign.

See these [Aerospike examples](https://github.com/aerospike/geospatial-samples).

## Geospatial Data

Aerospike supports the GeoJSON geospatial data type. All geospatial functionality (indexing and querying) only execute on GeoJSON data types.

GeoJSON data incurs this additional processing on data reads: 

- GeoJSON text is parsed for validity and support (see [GeoJSON Parsing](#geojson-parsing)).
- GeoJSON text is converted into S2 CellID coverings.
- Aerospike saves both the covering CellIDs and the original GeoJSON in the database. 
- Only GeoJSON data is accessible to the application through the client APIs and the UDF subsystem.

## Geospatial Index

In addition to integers and strings, Aerospike supports [Geo2DSphere](http://api.mongodb.org/csharp/current/html/M_MongoDB_Driver_IndexKeysDefinitionBuilder_1_Geo2DSphere_1.htm) data types for indexes. 

This example aql script creates a Geo2DSphere data type index:

```bash
aql> CREATE INDEX my_geo_index ON test.demoset (geobin) GEO2DSPHERE
```
Geo2DSphere indexes behave as other index types to:

- Scan existing records to inspect the indexed bin (`geobin` in the example above) to build an in-memory geospatial index.
- Create an independent index for data on each node.
- Update the index on all subsequent data inserts and updates.
- Rebuild the index when a node restarts.

## Geospatial Query

Aerospike supports two Geospatial queries:

- Points within a region (including circle)
- Region contains point

### Points-within-Region&mdash;Circle Query

This example to Python script is a points-within-a-region query.

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
    predicate = aerospike.predicates.geo_within_geojson_region(LOCBIN, region.dumps())
    query.where(predicate)
    
    """Define callback to process query result."""
    def callback((key, metadata, record)):
    	records.append(record)

    """Make the actual query!"""

    query.foreach(callback)

```


Example for points-within-a-region query using AQL.

* Insert a point data into aerospike.

```bash
aql> INSERT INTO test.testset (PK, geo_query_bin) VALUES (2, GEOJSON('{"type": "Point", "coordinates": [1,1]}'))
```

* Querying a region to see if it contains that point.

```bash
aql> SELECT * FROM test.testset WHERE geo_query_bin CONTAINS GeoJSON('{"type":"Polygon", "coordinates": [[[0,0], [0, 10], [10, 10], [10, 0], [0,0]]]}'))
+----------------------------------------------------+
| geo_query_bin                                      |
+----------------------------------------------------+
| GeoJSON('{"type": "Point", "coordinates": [1,1]}') |
+----------------------------------------------------+
1 row in set (0.004 secs)
```


### Region-Contains-Points Query

This example C++ script is a region-contains-points query.

```
// Callback function to process each record response
bool
query_cb(const as_val * valp, void * udata)
{
	if (!valp)
		return true;	// query complete

	char const * valstr = NULL;

	as_record * recp = as_record_fromval(valp);
	if (!recp)
		fatal("query callback returned non-as_record object");
	valstr = as_record_get_str(recp, g_valbin);

	__sync_fetch_and_add(&g_numrecs, 1);
	
	cout << valstr << endl;

	return true;
}

// Main query function
void
query_prcp(aerospike * asp, double lat, double lng)
{
	char point[1024];
    
    // Construct a GeoJSON point.
	snprintf(point, sizeof(point),
			 "{ \"type\": \"Point\", \"coordinates\": [%0.8f, %0.8f] }",
			 lng, lat);

    // Construct the query object.
	as_query query;
	as_query_init(&query, g_namespace.c_str(), g_set.c_str());

	as_query_where_inita(&query, 1);
	as_query_where(&query, g_rgnbin, as_geo_contains(point));

    // Make the actual query.
	as_error err;
	if (aerospike_query_foreach(asp, &err, NULL,
								&query, query_cb, NULL) != AEROSPIKE_OK)
		throwstream(runtime_error,
					"aerospike_query_foreach() returned "
					<< err.code << '-' << err.message);

	as_query_destroy(&query);
}
```

Example for region-contains-points query using AQL.

* Insert a region data into aerospike.

```bash
aql> INSERT INTO test.testset (PK, geo_query_bin) VALUES (1, GEOJSON('{"type": "Polygon", "coordinates": [[[0,0], [0, 10], [10, 10], [10, 0], [0,0]]]}'))
```

* Query a point in that region.

```bash
AQL> SELECT * FROM test.testset WHERE geo_query_bin CONTAINS GeoJSON('{"type":"Point", "coordinates": [1, 1]}')
+---------------------------------------------------------------------------------------------+
| geo_query_bin                                                                               |
+---------------------------------------------------------------------------------------------+
| GeoJSON('{"type": "Polygon", "coordinates": [[[0,0], [0, 10], [10, 10], [10, 0], [0,0]]]}') |
+---------------------------------------------------------------------------------------------+
1 row in set (0.017 secs)
```

<br>
## Query Filters

To extend the capabilities of both queries, use UDFs to filter the result set. 

This example Python script demonstrates using a filter UDF.

```python
def query_circle(args, client):
    """Query for records inside a circle."""
    query = client.query(args.nspace, args.set)
    predicate = aerospike.predicates.geo_within_radius(LOCBIN,
                                                       args.longitude,
                                                       args.latitude,
                                                       args.radius)
    query.where(predicate)
 
    # Search with UDF amenity filter
    query.apply('filter_by_amenity', 'apply_filter', [args.amenity,])
    query.foreach(print_value)
```

Where the `apply_filter` Lua function is the following: 

```
local function select_value(rec)
  return rec.val
end
 
function apply_filter(stream, amen)
  local function match_amenity(rec)
    return rec.map.amenity and rec.map.amenity == amen
  end
  return stream : filter(match_amenity) : map(select_value)
end
```

## Index on list/map

It is also possible to index and query on list/map elements with GeoJSON data type -

```
    # create a secondary index for numeric values of test.demo records whose 'points' bin is a list of GeoJSON points
    client.index_list_create('test', 'demo', 'points', aerospike.INDEX_GEO2DSPHERE, 'demo_point_nidx')

    predicate = aerospike.predicates.geo_within_radius('points',
                                                       args.longitude,
                                                       args.latitude,
                                                       args.radius,
                                                       aerospike.INDEX_GEO2DSPHERE)

    query = client.query('test', 'demo')
    query.where(predicate);

```

The above will create an index on geoJSON list elements, and construct the query predicate using the index.


## Aerospike GeoJSON Extension

Use the Aerospike `AeroCircle` geometry object to store circles along with regular polygons.

This example script specifies a circle with a radius of 300 meters at longitude/latitude -122.250629, 37.871022.

```bash
{"type": "AeroCircle", "coordinates": [[-122.250629, 37.871022], 300]}
```

## GeoJSON Parsing

On data insert/update, Aerospike only recognizes `Point`, `Polygon`, `MultiPolygon`, and `AeroCircle` [GeoJSON geometry objects](http://geojson.org/geojson-spec.html#geometry-objects), which are indexable objects. Unsupported GeoJson objects return an `AS_PROTO_RESULT_FAIL_GEO_INVALID_GEOJSON` error (for example, `LineString` or `MultiLineString` fail on insert). Holes can be `Polygon` objects, per the [GeoJSON Format Specification](http://geojson.org/geojson-spec.html#polygon).

{{#note}}
`Polygon` loop definitions must wind counter-clockwise. 
{{/note}}

Aerospike supports the `Feature` operator, which allows groups of geometry objects and user-specified properties; however, `Feature Collection` is not supported.

Invalid GeoJSON objects are caught on insert/update. For example, an object defined as `point` instead of `Point` fails. 

Per the GeoJSON IETF recommendation, the Coordinate System is WGS84. Explicit specification of CRS is ignored.

## Known Limitations

- Insert/update of GeoJSON data types is not supported using UDFs.
- Duplicate records can be returned.

## Create a Geospatial Application

To develop a geospatial application:

1. [Install](/docs/operations/install/index.html) and configure the [Aerospike server](https://www.aerospike.com/download).
1. Create a Geo2DSphere index on a namespace-set-bin combination.
1. Construct and insert GeoJSON `Point` data.
1. Construct a Points-within-Region predicate (`where` clause), make a query request, and process the records returned.
 1. (alternate) Construct and insert GeoJSON `Polygon`/`MultiPolygon` data.
 1. (alternate) Construct a Region-contains-Point predicate, make a query request, and process the records returned.
