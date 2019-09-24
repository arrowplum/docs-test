---
title: Map-Reduce Example - Aggregation Input
description: Aggregation Input Map-Reduce Example using Aerospike Connector
---

In this example, we read `test:integers` data from Aerospike. 
Data was generated using SampleData.java. 
Source code of the data generator can be reviewed here: 

```asciidoc
~/aerospike-hadoop/sampledata/src/main/java/com/aerospike/hadoop/sampledata/SampleData.java
```

It writes data starting and ending at the specified offset in the arguments. 
The integer value increments sequentially in `bin1`. `bin2` always has the string value = “value2”. 
The Primary Key  = ‘key-nnn” where nnn is the integer value of the offset. 

In the mapper phase, the key used is modulo 3163 of the value. 
In the reduce phase, the aggregations are done with respect to this key. 
In that sense, the example is rather contrived but is a good reference for dealing with integer data on Aerospike.

For example, using AQL, we can inspect the source data as follows.

```asciidoc
aql> select * from test.integers where PK = 'key-986'
+------+----------+
| bin1 | bin2     |
+------+----------+
| 986  | "value2" |
+------+----------+
1 row in set (0.000 secs)
```

The aggregation example computes `num`, `sum`, `min`, `max` on `bin1` data. The aggregation example source code can be viewed here:
 
```asciidoc
~/aerospike-hadoop/examples/aggregate_int_input/src/main/java/com/aerospike/hadoop/examples/aggregateintinput/AggregateIntInput.java
```
<center>
{{#figure "" "Aggregation Input Example" size="large" }}
<img src="/docs/connectors/assets/images/AggInputExample.png">
{{/figure}}
</center>

Execute the example as follows. First clean out the HDFS output directory from previous runs.

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ hdfs dfs -rm -R /tmp/output
15/06/29 16:22:22 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /tmp/output

hdclient@ztg-client:~/aerospike-hadoop$ hadoop jar ./examples/aggregate_int_input/build/libs/aggregate_int_input.jar -D aerospike.input.host=ztg-client -D aerospike.input.namespace=test -D aerospike.input.setname=integers -D aerospike.input.binnames=bin1,bin2 -D aerospike.input.operation=numrange -D aerospike.input.numrange.bin=bin1 -D aerospike.input.numrange.begin=100 -D aerospike.input.numrange.end=200 /tmp/output
```

Check the output:

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ hdfs dfs -cat /tmp/output/part-r-00000 
100    1 100 100 100
101    1 101 101 101
102    1 102 102 102
103    1 103 103 103
104    1 104 104 104
105    1 105 105 105
```

**Note:** Since we are taking only 101 rows of data `(start=100, end=200)`, modulo with 3163 results in a unique key for each row hence, in each key, the reducer outputs number of data points `(num) =1, min = max = sum = key`. 
However, if the example is repeated by changing the source code with modulo 1 instead of 3163, for the same example, it will yield:

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ hdfs dfs -cat /tmp/output/part-r-00000 
0	101 100 200 15150
```
ie. `key=0` always (modulo 1 is always 0), `num = 101` data points, `min=100, max=200` and `sum = 15150`.



