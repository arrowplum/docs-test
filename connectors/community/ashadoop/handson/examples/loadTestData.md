---
title: Load Test Data
description: Load Test Data for Aerospike Hadooop Connector Usage Examples
---
The figure below shows the data we will load to run our sample examples.

<center>
{{#figure "" "Load Test Data for Examples" size="large" }}
<img src="/docs/connectors/assets/images/LoadTestData.png">
{{/figure}}
</center>

- Load `test:words` data into Aerospike using `sampledata.jar`.

- As user `hdclient` on `ztg-client`, take any log file and copy it to `~/tmp/input` in the local file system. (We used the Aerospike server log file.)

```asciidoc
hdclient@ztg-client:~$ ls
aerospike-hadoop  hadoop  hadoop-2.6.0.tar.gz

hdclient@ztg-client:~$ mkdir tmp

hdclient@ztg-client:~$ cp /var/log/aerospike/aerospike.log ~/tmp/input
```
Check...

```asciidoc
hdclient@ztg-client:~$ tail -2 ./tmp/input
Jun 29 2015 19:15:17 GMT: INFO (info): (hist.c::137) histogram dump: query (0 total) msec
Jun 29 2015 19:15:17 GMT: INFO (info): (hist.c::137) histogram dump: query_rec_count (0 total) count
hdclient@ztg-client:~$
```

- Copy to `ztg-master` to later load into HDFS on the hadoop cluster.

```asciidoc
hdclient@ztg-client:~$ scp /home/hdclient/tmp/input hdclient@ztg-master:/home/hdclient/tmp
input                 100% 4581KB   4.5MB/s   00:00    
hdclient@ztg-client:~$
```

- On `ztg-client`, use `sample.jar` to load `~/tmp/input` to Aerospike as `test:words`. 
(Aerospike server must be running on `ztg-client`.)  Each line of the log file is loaded as a text value in `bin1` of Aerospike. 

```asciidoc
hdclient@ztg-client:~$ cd aerospike-hadoop/sampledata
hdclient@ztg-client:~/aerospike-hadoop/sampledata$ java -jar ./build/libs/sampledata.jar ztg-client:3000:test:words:bin1 text-file ~/tmp/input
2015-06-29 12:43:36.015 INFO  SampleData:132 - starting
2015-06-29 12:43:36.022 INFO  SampleData:56 - saw ztg-client:3000:test:words:bin1 text-file
2015-06-29 12:43:36.058 INFO  SampleData:87 - processing /home/hdclient/tmp/input ...
2015-06-29 12:43:39.033 INFO  SampleData:97 - inserted 37400 records
2015-06-29 12:43:39.033 INFO  SampleData:134 - finished
```

Inspect `SampleData.java` code provided by Aerospike in the connector samples. It uses `path:linenum` as the primary key in Aerospike. 
The path in the above example is `/home/hdclient/tmp/input` and its use is demonstrated below.

Check using AQL:

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop/sampledata$ aql
Aerospike Query
Copyright 2013 Aerospike. All rights reserved.

aql> show sets
+-----------+----------------+----------------------+---------+----------+------------+---------------------+
| n_objects | set-enable-xdr | set-stop-write-count | ns_name | set_name | set-delete | set-evict-hwm-count |
+-----------+----------------+----------------------+---------+----------+------------+---------------------+
| 37400     | "use-default"  | 0                    | "test"  | "words"  | "false"    | 0                   |
+-----------+----------------+----------------------+---------+----------+------------+---------------------+
1 row in set (0.000 secs)
OK
aql> select * from test.words where PK = '/home/hdclient/tmp/input:1'
+-----------------------------------------------------------------------------------------------------------------------------------------------+
| bin1                                                                                                                                          |
+-----------------------------------------------------------------------------------------------------------------------------------------------+
| "Jun 29 2015 13:35:23 GMT: INFO (info): (thr_info.c::4804)  migrates in progress ( 0 , 0 ) ::: ClusterSize 1 ::: objects 0 ::: sub_objects 0" |
+-----------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.001 secs)

aql> exit
```

- On `ztg-client`, use the command below to fill Aerospike server 
(must be running already) with generated sequential integers data for `aggregate_int_input` demo.

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop/sampledata$ java -jar build/libs/sampledata.jar ztg-client:3000:test:integers:bin1 seq-int 0 100000
2015-06-29 12:50:41.515 INFO  SampleData:132 - starting
2015-06-29 12:50:41.521 INFO  SampleData:56 - saw ztg-client:3000:test:integers:bin1 seq-int
2015-06-29 12:50:42.571 INFO  SampleData:113 - created secondary index on bin1
2015-06-29 12:50:50.072 INFO  SampleData:126 - inserted 100000 records
2015-06-29 12:50:50.072 INFO  SampleData:134 - finished
hdclient@ztg-client:~/aerospike-hadoop/sampledata$

Check using AQL:

aql> show sets
+-----------+----------------+----------------------+---------+------------+------------+---------------------+
| n_objects | set-enable-xdr | set-stop-write-count | ns_name | set_name   | set-delete | set-evict-hwm-count |
+-----------+----------------+----------------------+---------+------------+------------+---------------------+
| 37400     | "use-default"  | 0                    | "test"  | "words"    | "false"    | 0                   |
| 100000    | "use-default"  | 0                    | "test"  | "integers" | "false"    | 0                   |
+-----------+----------------+----------------------+---------+------------+------------+---------------------+
2 rows in set (0.000 secs)
OK

aql> exit
```

- Moving `ztg-master:/home/hdclient/tmp/input` text data into HDFS using user `hdclient`.
We must update access permissions of `/tmp` so that `hdclient` can access it.
Log into `ztg-master` as user `hduser` (of supergroup) and change its permissions as follows:

```asciidoc
hdfs dfs -chmod 777 /tmp 
hduser@ztg-master:~$ hdfs dfs -ls /
Found 2 items
drwxrwxrwx   - hduser supergroup          0 2015-06-29 13:14 /tmp
drwxr-xr-x   - hduser supergroup          0 2015-06-26 18:27 /user
```

- As `hdclient` on `ztg-client`, move `~/tmp/input` to `HDFS:/tmp/words`.

Delete `hdfs:/tmp/words` if it already exists.

```asciidoc
hdclient@ztg-client:~/hadoop$ hdfs dfs -rm /tmp/words
15/06/29 14:11:29 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /tmp/words

hdclient@ztg-client:~/hadoop$ hdfs dfs -copyFromLocal ~/tmp/input /tmp/words
hdclient@ztg-client:~/hadoop$ hdfs dfs -ls /tmp
Found 2 items
drwx------   - hduser   supergroup          0 2015-06-25 22:11 /tmp/hadoop-yarn
-rw-r--r--   3 hdclient supergroup    4690640 2015-06-29 14:12 /tmp/words
```

We have successfully loaded all the test data we need to run our examples in the various places per the figure above.


