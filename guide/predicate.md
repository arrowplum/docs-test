---
title: Predicate Filtering
description: Use Predicate Filtering to scope down record sets.
assets: /docs/guide/assets
---
<div style="float: right" >
![]({{book.assets}}/icon-query-2.png)
</div>

Predicate filtering is introduced in Aerospike Server 3.12. 
Predicate expressions provide a mechanism for additional filtering on the resultset of a secondary index query or scan operation.

Predicate expressions use record metadata and record bin values. They can be built from -
* Record storage size
* Record last update time
* Record expiration time
* Record digest modulo
* Integer, string and GeoJSON bin values
* List and map element values

Predicate operators include -
* Logical AND, OR and NOT expressions
* Integer <, <=, ==, !=, >=, and >
* String == and !=
* GeoJSON "within" and "contains" comparisons
* String regular expressions

## Design Considerations

When multiple criteria are being considered for a query, consider choosing the most selective criterium (fewest results) and build a secondary index using it, to narrow down the selectivity. Then add the remaining criteria as predicate expressions. If not, the predicate expression is applied to all records in a scan.

For example, to find all users who has last logged in the past week and lives in Sunnyvale, one should consider first building a secondary index based on a "login_time". A predicate expression of "last-week < login_time < now AND location=Sunnyvale" will be able to quickly get the batch of users based on login_time first.

Some of the predicate expression evaluates on record metadata. When a predicate expression can reject a record based on metadata, it will skip fetching the record from storage.  Such expressions are always favored, as they can be extremely fast, particularly when the records are stored in an SSD configuration.

When applied in conjunction with RecordUDF (both scan and secondary index query), predicate expressions are evaluated before the UDF operation, so UDF operation can be bypassed.

When applied in conjunction with scan with "nobin" policy on, any bin-level predicate filter will be ignored, since record data is not fetched.

Note that data types are always explicitly expressed. Type coersion is not allowed (for example, comparing a double against an integer). 

Error or unknown conditions are evaluated to false. For example, a mismatch of data type, or evaluating a non-existing bin, both evaluates to false. 

## Use Cases 

Common applications of filtering can be:

- Scan for all records with stored record size above X.
- Filter for a user segment match for all users with activity within a timestamp range.
- Filter for a specific amenity after a points-within-region Geo query.
- Uniquely divide a scan workload amongst N clients via a digest_modulo(N) filter.

## Limitations
- Predicate Filter is not applied to the following transaction types - batch, aggregation, single-transaction.
- Record storage size filter is non-zero only for records in namespace with persistence.

## Defining Predicate Filter - Postfix API

Predicate expression sequences use postfix notation (AKA Reverse Polish Notation (RPN)) and an expression stack to generate a corresponding expression tree.

A valid expression sequence must reduce to a single expression tree. When evaluated, it must resolve to one single true/false value, indicating if the record should be returned.

As an example, the following predicate expression sequence is added onto a secondary index query. The secondary index query first narrows the resultsets to records with bin "id" equals to string value "abc". Then the resultset is predicate filtered for "age>=30" and "age<=40" and "zone_id=3". 

This expression has "and" as the top level logical expression, which is parametrized to consume 3 logical values (true/false) resulted through each of the 3 sub-tree evaluation. After evaluating the expression, it will have a single logical value (true/false)

```c
    as_query q;
    as_query_init(&q, NAMESPACE, SET);

    as_query_where_inita(&q, 1);
    as_query_where(&q, "id", as_string_equals("abc"));

    as_query_predexp_inita(&q, 10);
    as_query_predexp_add(&q, as_predexp_integer_bin("age"));
    as_query_predexp_add(&q, as_predexp_integer_value(30));
    as_query_predexp_add(&q, as_predexp_integer_greatereq()); // Consumes bin("age") and "30" and produces true/false
 
    as_query_predexp_add(&q, as_predexp_integer_bin("age"));
    as_query_predexp_add(&q, as_predexp_integer_value(40));
    as_query_predexp_add(&q, as_predexp_integer_lesseq());    // Consumes bin("age") and "40" and produces true/false
   
    as_query_predexp_add(&q, as_predexp_integer_bin("zone_id"));
    as_query_predexp_add(&q, as_predexp_integer_value(3));
    as_query_predexp_add(&q, as_predexp_integer_equal());     // Consumes bin("zone_id") and "3" and produces true/false

    as_query_predexp_add(&q, as_predexp_and(3));              // Consumes the 3 true/false already on stack and produces true/false

    aerospike_query_foreach(as, &err, NULL, &q, callback, &datap);
```

Below is a categorization of the expressions based on their stack consumption behavior 

The following predicate expression nodes push a single "value" expression onto the expression stack:

* as_predexp_integer_value(int64_t value);
* as_predexp_string_value(char const * value);
* as_predexp_geojson_value(char const * value);
* as_predexp_integer_bin(char const * binname);
* as_predexp_string_bin(char const * binname);
* as_predexp_geojson_bin(char const * binname);
* as_predexp_list_bin(char const * binname);
* as_predexp_map_bin(char const * binname);
* as_predexp_integer_var(char const * varname);
* as_predexp_string_var(char const * varname);
* as_predexp_geojson_var(char const * varname);
* as_predexp_rec_device_size();
* as_predexp_rec_last_update();
* as_predexp_rec_void_time();

The following predicate expression nodes pop two "value" expressions from the stack and push a single "logical" expression in their place:

* as_predexp_integer_equal();
* as_predexp_integer_unequal();
* as_predexp_integer_greater();
* as_predexp_integer_greatereq();
* as_predexp_integer_less();
* as_predexp_integer_lesseq();
* as_predexp_string_equal();
* as_predexp_string_unequal();
* as_predexp_string_regex(uint32_t opts);
* as_predexp_geojson_within();
* as_predexp_geojson_contains();

The following predicate expression nodes pop one or more "logical" expressions from the stack and push a single "logical" expression in their place:

* as_predexp_and(uint16_t nexpr);
* as_predexp_or(uint16_t nexpr);
* as_predexp_not();

The following predicate expression nodes pop one "logical" expression and one "value" expression from the stack and push a single "logical" expression in their place:

* as_predexp_list_iterate_or(char const * varname);
* as_predexp_list_iterate_and(char const * varname);
* as_predexp_mapkey_iterate_or(char const * varname);
* as_predexp_mapkey_iterate_and(char const * varname);
* as_predexp_mapval_iterate_or(char const * varname);
* as_predexp_mapval_iterate_and(char const * varname);

For more details on the RPN styled API, please see [C Client API](https://github.com/aerospike/aerospike-client-c/blob/master/src/include/aerospike/as_predexp.h)    
