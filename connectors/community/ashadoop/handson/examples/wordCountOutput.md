---
title: Map-Reduce Example - Word Count Output
description: Word Count Output Map-Reduce Example using Aerospike Connector
---

### Word Count - Output
In this example, we will read `/tmp/input` (log text data) in HDFS, do a word count, 
but instead of putting the results in HDFS, push them to `test:counts` on Aerospike. 
We will achieve this by setting the output format class in the map-reduce [job driver](/docs/connectors/community/ashadoop/handson/examples/jobDriver.html) 
to an extension of the `AerospikeOutputFormat` class. Since the original data is same as the [Word Count Input Example](/docs/connectors/community/ashadoop/handson/examples/wordCountInput.html),
we expect same results.

<center>
{{#figure "" "Word Count Output Example" size="large" }}
<img src="/docs/connectors/assets/images/WordCountOutputExample.png">
{{/figure}}
</center>

`test:counts` has two bins, `word` and `count`.
The `word` bin value is also the Primary Key for that record and 
can be used in the AQL select statements to examine individual records.

Source code can be reviewed here: 

```asciidoc
~/aerospike-hadoop/examples/word_count_output/src/main/java/com/aerospike/hadoop/examples/wordcountoutput/WordCountOutput.java
```

Running the test:

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ cd ~
hdclient@ztg-client:~$ cd aerospike-hadoop/
hdclient@ztg-client:~/aerospike-hadoop$ hadoop jar ./examples/word_count_output/build/libs/word_count_output.jar -D aerospike.output.namespace=test -D aerospike.output.setname=counts /tmp/words

Inspect result in Aerospike using AQL:
hdclient@ztg-client:~/aerospike-hadoop$aql
aql> select * from test.counts where PK = 'writes_master'
+-----------------+-------+
| word            | count |
+-----------------+-------+
| "writes_master" | 2040  |
+-----------------+-------+
1 row in set (0.000 secs)

aql> select * from test.counts where PK = '{bar}'
+---------+-------+
| word    | count |
+---------+-------+
| "{bar}" | 340   |
+---------+-------+
1 row in set (0.001 secs)

aql> exit

hdclient@ztg-client:~/aerospike-hadoop$

The results for counts are same as in the Word Count Input Example.
```

