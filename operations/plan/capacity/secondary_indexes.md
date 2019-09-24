---
title: Secondary Index Capacity Planning
description: The resources required by Aerospike secondary indexes
---

### Memory Required

**For cardinality < 32** (i.e. less than 32 records indexed per secondary index key)

|Type|Min. mem usage|Max. mem usage|Avg. case mem usage|
|---|---|---|---|
|Integer|(16.25 x 1 x K) + (20 x 1 x R)|(16.25 x 2 x K) + (20 x 3.5 x R)|(16.25 x 1.5 x K) + (20 x 2 x R)|
|String|(28.44 x 1 x K) + (20 x 1 x R)|(28.44 x 2 x K) + (20 x 3.5 x R)|(28.44 x 1.5 x K) + (20 x 2 x R)|

**For cardinality > 32** (i.e. more than 32 records indexed per secondary index key)

|Type|Min. mem usage|Max. mem usage|Avg. case mem usage|
|---|---|---|---|
|Integer|(16.25 x 1 x K) + (28.44 x 1 x R)|(16.25 x 2 x K) + (28.44 x 2 x R) + (28.44 x K)|(16.25 x 1.5 x K) + (28.44 x 1.5 x R)|
|String|(28.44 x 1 x K) + (28.44 x 1 x R)|(28.44 x 2 x K) + (28.44 x 2 x R) + (28.44 x K)|(28.44 x 1.5 x K) + (28.44 x 1.5 x R)|

- Extreme values may vary by a few hundred KBs.
- Secondary index garbage collection may affect this value by a few hundred MBs too.
- K - number of secondary index keys in the secondary index.
- R - number of records/objects indexed in the secondary index.
- Each formula can be divided as

        Sum of (m * a * n) for n = keys and objects + constant
        n - number of keys/records in secondary index.
        m - memory overhead per key/record in a btree or array.
        a - memory allocated per memory used by secondary index.

        Following tables explains the different values of m and a.

|Keys/Objects|Type|Cardinality|Value&nbsp;of&nbsp;m|Note|
|---|---|---|---|---|
|Keys|Integer|Any|16.25 |8 bytes for integer + 8 bytes for pointer + 8 /31 (one extra pointer / btree degree)|
|Keys|String|Any|28.44|20 bytes digests + 8 bytes pointer + 8 /18 (one extra pointer / btree degree)|
|Records/objects|Any|< 32|20|For cardinality < 32, arrays are used to store objects. Over head is the size of digest (20)|
|Records/objects|Any|> 32|28.44|For cardinality > 32, objects are are stored in btree (same as string type keys)|

|Scenario|Keys/Objects|Cardinality|Value&nbsp;of&nbsp;a|Note|
|---|---|---|---|---|
|Best Case|Any|Any|1|In the best case scenario every memory which is allocated is being used.|
|Worst case|Keys|Any|2|Keys are always stored in btree. In the worst case every node of btree is half full.|
|Worst case|Records/objects|< 32|3.5|For cardinality < 32, arrays are used to store objects. In the worst case 3.5 times memory is allocated extra.|
|Worst case|Records/objects|> 32|2|For cardinality > 32, objects are stored similarly as keys.|
|Avg case|Keys|Any|1.5|Generally, memory efficiency of btree lies between 0.5 and 1.|
|Avg case|Records/objects|< 32|2|For cardinality < 32, objects are stored in arrays. Generally they are allocated twice more than needed.|
|Avg case|Records/objects|> 32|1.5|Same as average case for keys.|

    Note - In the worst case, each btree can also have an extra allocated unused btree node.
    Thus (28.44 * K) is added into the calculations.

- Why 18 and 31 - We chose fan out of integer type btree as 31 and string/digest type btree as 18 because of the following reasons:
 - To make the size of each node of btree lie in multiples of cache-line (64 bytes).
 - To make the size of nodes of each type of btree similar. (512 bytes)

For string/digest type btree span out comes as:

    (512 - 8 byte header)/(20 byte string digest + 8 byte pointer to child node) = 18 entries

For integer type btree span out comes as:

    (512 - 8 byte header)/(8 byte integer + 8 byte pointer) = 31 entries


{{#note markdown=true}}For secondary indexes, not all records are indexed — only records containing a specific bin will be indexed. Records that do not contain the indexed bin are ignored — that is, Aerospike can’t use an index to count or return information for records that do not include the specified bin. For example, if only half of your data includes a particular bin that is being indexed, then the resulting index will be only half the size of another index for which all of your data includes the indexed data.

**Please [contact us](/contact/) for any further inquiries regarding secondary index capacity planning.**{{/note}}

