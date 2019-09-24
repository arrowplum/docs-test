## Word count input from Aerospike

Read text data from records in Aerospike and count unique words in Hadoop, output results to HDFS.

**Sample output in HDFS:** 
```asciidoc
$hadoop fs -cat /tmp/output/part*|more
--A    1
--And  2
--But  1
```
[Explore this example further.](/docs/connectors/community/ashadoop/handson/examples/wordCountInput.html)

## Word count output to Aerospike
Read text data from file in HDFS, count unique words in Hadoop, output results to Aerospike.

**Sample output:**  `aql> select * from test.counts`  
```asciidoc
+----------------+-------+
| word       	| count |
+----------------+-------+
| "bowsy"    	| 1 	|
| "heel."    	| 1 	|
| "anything."	| 2 	|
```

[Explore this example further.](/docs/connectors/community/ashadoop/handson/examples/wordCountOutput.html)

## Aggregating Integers 
Read integer data from records in Aerospike and aggregate by key (arbitrarily chosen to be  modulo 3163 of the value) - find count, min, max and sum for each key.  Write aggregation results to HDFS.

**Sample output:** `hadoop fs -cat /tmp/output/part*|more`  
```asciidoc
0    32 0  98053 1568848
6    32 6  98059 1569040
12   32 12 98065 1569232
```

[Explore this example further.](/docs/connectors/community/ashadoop/handson/examples/aggInput.html)


## Sessionization & Joining

This example comprises analyzing web server log files for identifying sessions, creating user profiles and joining example.

This is a three part example. It uses web server log data stored in HDFS. In step 1, the log data is sessionized and output to Aerospike. In step 2, userids in the web server logs are assigned a profile (age and gender, generated randomly). Finally in step 3, sessionization data is again extracted from web server logs (just like step 1) and then joined with userid profile data read from Aerospike, the joined data is re-written back to Aerospike into a new set.

### Sessionization
Typical log line entries as shown below are used to extract session information.  
```
946628 - - [15/Jun/1998:22:00:01 +0000] "GET /images/blank.gif HTTP/1.1" 200 85
```
**Sample output:** `select * from test.sessions`  
```asciidoc
+---------+--------------+--------------+-------+
| userid  | start    	| end      	| nhits |
+---------+--------------+--------------+-------+
| 1593760 | 897967790000 | 897968633000 | 131   |
| 1591365 | 897948422000 | 897949662000 | 195   |
```

### Creating user profiles
Extract unique userid from log file data in HDFS, assign random age and gender	and write to Aerospike.

**Sample output:** `select * from test.profiles`  
```asciidoc
+---------+-----+--------+
| userid  | age | isMale |
+---------+-----+--------+
| 1520488 | 28  | 0  	|
| 1597667 | 47  | 1  	|
```

### External Join
Sessionize same as part 1, read profile data from part 2, join the two and write result to new set in Aerospike.  
**Sample output:** `select * from test.sessions`  
```asciidoc
+---------+--------------+--------------+-------+
| userid  | start    	| end      	| nhits |
+---------+--------------+--------------+-------+
| 1593760 | 897967790000 | 897968633000 | 131   |
| 1591365 | 897948422000 | 897949662000 | 195   |
| 1261950 | 897982435000 | 897982608000 | 70	|
```


[Explore this example further.](/docs/connectors/community/ashadoop/handson/examples/externalJoin.html)

Before we can get into building and running these examples from the source code, we need a standalone Apache Hadoop multi-node cluster, accessible from an edge node. In the following sections, we will achieve this topology and configuration.
