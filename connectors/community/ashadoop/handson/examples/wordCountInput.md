---
title: Map-Reduce Example - Word Count Input
description: Word Count Input Map-Reduce Example using Aerospike Connector
---

### Word Count - Input

In this example, we read `test:words` data from Aerospike, do a word count on the data and write the output to `HDFS` in `/tmp/output`. 

`test:words` data is organized by Primary Key = `path:linenum` where in our example path is `/home/hdclient/tmp/input`. 
In our example, each line is contained in `bin1`. A line is read and tokenized into words before doing the word count.

<center>
{{#figure "" "Word Count Input Example" size="large" }}
<img src="/docs/connectors/assets/images/WordCountInputExample.png">
{{/figure}}
</center>

**Source code:**
```asciidoc
~/aerospike-hadoop/examples/word_count_input/src/main/java/com/aerospike/hadoop/examples/wordcountinput/WordCountInput.java
```
#### Steps to run the example:

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ cd ~
hdclient@ztg-client:~$ cd aerospike-hadoop/
hdclient@ztg-client:~/aerospike-hadoop$ hadoop jar ./examples/word_count_input/build/libs/word_count_input.jar -D aerospike.input.namespace=test -D aerospike.input.setname=words -D aerospike.input.operation=scan /tmp/output
15/06/29 14:27:08 INFO wordcountinput.WordCountInput: run starting
. . .
. . .
15/06/29 14:27:14 INFO wordcountinput.WordCountInput: finished

hdclient@ztg-client:~/aerospike-hadoop$ hdfs dfs -ls /tmp/output

Found 2 items
-rw-r--r--   3 hdclient supergroup          0 2015-06-29 14:27 /tmp/output/_SUCCESS
-rw-r--r--   3 hdclient supergroup      52510 2015-06-29 14:27 /tmp/output/part-00000

hdclient@ztg-client:~/aerospike-hadoop$ hdfs dfs -cat /tmp/output/part-00000

(	4080
. . .
. . .
writes_master	2040
writes_reply	2040
zombie	2040
{bar}	340
{test}	340
```

